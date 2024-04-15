# Chapter 4
## Case study ER injuries

Đầu tiên load các library cần dùng trong chapter này
```
# Load library
library(shiny)
library(vroom)
library(tidyverse)
```

Tạo folder để download data vào, đoạn code này dùng `dir.create` sẽ tạo folder vào trong directory của chúng ta (kiểm tra directory bằng `getwd`)

```
# Tạo folder ten neiss
dir.create("neiss")

# Download 
download <- function(name) {
  url <- "https://github.com/hadley/mastering-shiny/raw/main/neiss/"
  download.file(paste0(url, name), paste0("neiss/", name), quiet = TRUE)
}
download("injuries.tsv.gz")
download("population.tsv")
download("products.tsv")

injuries <- vroom::vroom("neiss/injuries.tsv.gz")
products <- vroom::vroom("neiss/products.tsv")
population <- read_tsv("neiss/population.tsv")
```
Lưu ý: code này đã được chỉnh sửa, url trên sách gốc đã bị thay đổi

Đầu tiên phải mở dataset chính injuries để xem cấu trúc data
```
injuries
```
Trong đó ta có cột:
- `trmt_date` là ngày mà bệnh nhân nhập viện
- `body_part` là phần cơ thể bị tai nạn
- `diag` là mã chẩn đoán
- `prod_code` là cơ chế chấn thương (do dao, dụng cục bếp...)
- `weight` weight của số người ước tính bị chấn thương tương tự scale với toàn bộ dân số Mỹ
- `narative` sơ lược bệnh sử nhập viện

Ngoài ra còn có data population, phân phối dân số theo nhóm tuổi ở Mẽo năm 2017
```
population
```
`products` Làm dictionnary cho `prod_code`
```
products
```

## Tìm hiểu data
Đầu tiên tìm hiểu data, ví dụ muốn xem data về các chấn thương chỉ xảy ra trong toilet. Ta có thể tạo riêng một bảng dữ liệu riêng về toilet injuries

```
selected <- injuries %>% filter(prod_code == 649)
nrow(toilet)
```

Dùng các phương pháp data exploration để hiểu rõ hơn về data
```
selected %>% count(location, wt = weight, sort = TRUE)
selected %>% count(location, sort = TRUE)
selected %>% count(body_part, wt = weight, sort = TRUE)
```

Để summary và tạo một chart thì ta cần data theo độ tuổi 
```
summary <- selected %>%
  count(age, sex, wt = weight)
```

Thử vẽ data
```
summary %>%
  ggplot(aes(age, n, color = sex)) +
  geom_line() +
  labs(y = "Estimated number of injuries")
```

Nhưng rõ ràng data này chưa được standardize theo độ tuổi. ta có thể dùng weight và population để standardize
```
summary <- selected %>% 
  count(age,sex,wt = weight) %>%
  left_join(population, by = c("age","sex")) %>%
  mutate(rate = n/population * 10000)

summary %>% 
  ggplot(aes(age, rate, colour = sex)) +
  geom_line(na.rm = TRUE) +
  labs (y = "Injuries per 10,000 people")
```

Double-check, kiểm tra lại xem giả thuyết của mình có chính xác chưa

```
selected %>% 
  sample_n(10) %>% 
  pull(narrative)
```
## Prototype
Đầu tiên khi tạo một prototype, nên chọn bắt đầu càng đơn giản nhất càng tốt.

Bắt đầu suy nghĩ về UI và các thứ cần nên hiển thị trong UI, load library và viết UI.

Ví dụ ta muốn tạo một shiny App với khả năng chọn dạng tai nạn (exp: toilet) thì sẽ xuất ra 3 bảng xếp thử tự weighting của chẩn đoán, phần cơ thể bị chấn thương, và hiện trường ở đâu.

```
library(shiny)
prod_codes <- setNames(products$prod_code, products$title)

ui <- fluidPage(
  fluidRow(
    column(6,
      selectInput("code", "Product", choices = prod_codes)
    )
  ),
# Mỗi table sẽ được 4 columns
  fluidRow(
    column(4, tableOutput("diag")),
    column(4, tableOutput("body_part")),
    column(4, tableOutput("location"))
  ),
# Tổng của 3 table 12 column 
  fluidRow(
    column(12, plotOutput("age_sex"))
  )
)
```
Sử dụng code server để rút các bảng và plot ra như ví dụ trên đã làm nhưng sẽ ở một topic rộng hơn 

```
server <- function(input, output, session) {
  selected <- reactive(injuries %>% filter(prod_code == input$code))

  output$diag <- renderTable(
    selected() %>% count(diag, wt = weight, sort = TRUE)
  )
  output$body_part <- renderTable(
    selected() %>% count(body_part, wt = weight, sort = TRUE)
  )
  output$location <- renderTable(
    selected() %>% count(location, wt = weight, sort = TRUE)
  )

  summary <- reactive({
    selected() %>%
      count(age, sex, wt = weight) %>%
      left_join(population, by = c("age", "sex")) %>%
      mutate(rate = n / population * 1e4)
  })

  output$age_sex <- renderPlot({
    summary() %>%
      ggplot(aes(age, n, colour = sex)) +
      geom_line() +
      labs(y = "Estimated number of injuries")
  }, res = 96)
}
```


## Làm đẹp các bảng
Thay vì hiển thị toàn bộ chiều dài của bản, có thể làm cho bảng chỉ hiện thị 5 dòng có giá trị cao nhất
Test thử trên script
```
injuries %>%
  mutate(diag = fct_lump(fct_infreq(diag), n = 5)) %>%
  group_by(diag) %>%
  summarise(n = as.integer(sum(weight)))
```

Tạo một function để tránh lặp lại khi code
```
injuries %>%
  mutate(diag = fct_lump(fct_infreq(diag), n = 5)) %>%
  group_by(diag) %>%
  summarise(n = as.integer(sum(weight)))
```

Chép đoạn output mới cho server
```
  output$diag <- renderTable(count_top(selected(), diag), width = "100%")
  output$body_part <- renderTable(count_top(selected(), body_part), width = "100%")
  output$location <- renderTable(count_top(selected(), location), width = "100%")
```
Dùng width = '100%' là để cho dù UI có 4 column và data chỉ có 2 column thì data vận có chiều rộng như 4 columns

## Chọn hiển thị plot khác nhau Rate/Count
Sử dụng UI để người dùng chọn dạng plot
```
  fluidRow(
    column(8,
      selectInput("code", "Product",
        choices = setNames(products$prod_code, products$title),
        width = "100%"
      )
    ),
    column(2, selectInput("y", "Y axis", c("rate", "count")))
  ),
```
Thay đổi output cho phù hợp
```
  output$age_sex <- renderPlot({
    if (input$y == "count") {
      summary() %>%
        ggplot(aes(age, n, colour = sex)) +
        geom_line() +
        labs(y = "Estimated number of injuries")
    } else {
      summary() %>%
        ggplot(aes(age, rate, colour = sex)) +
        geom_line(na.rm = TRUE) +
        labs(y = "Injuries per 10,000 people")
    }
  }, res = 96)
```

## Tạo reactive button cho narrative
Ví dụ muốn tạo thêm một nút reactive mỗi lần click vào sẽ ra bệnh sử của mỗi ca khác nhau 
Trên UI ta có thể sử dụng actionButton() mỗi lần click vào sẽ đưa input đến với server
```
  fluidRow(
    column(2, actionButton("story", "Tell me a story")),
    column(10, textOutput("narrative"))
  )
```

Trên server để nhận thông tin từ actionButtion
```
  narrative_sample <- eventReactive(
    list(input$story, selected()),
    selected() %>% pull(narrative) %>% sample(1)
  )
  output$narrative <- renderText(narrative_sample())
```










