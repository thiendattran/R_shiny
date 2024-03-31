# BASIC REACTIVITY
## MOT VAI LUU Y
${\color{red}Reactive \ programming \ }$(mo hinh phan ung): khi input thay doi thi output se tu dong thay doi
<br> **Reactivity** means that outputs automatically update as inputs change. 
<br> `ui` thi don gian vi moi `user` se nhan cung duong link HTML
<br> `server` thi phuc tap hon vi moi `user` can nhan version rieng cua APP
### CODE CHUONG 2
```
ui <- fluidPage(
  numericInput("count", label = "Number of values", value = 100)
)
```
`input` thi read-only. Neu co gang thay doi trong `server` function, no se bao `error`
```
server <- function(input, output, session) {
  input$count <- 10  
}

shinyApp(ui, server)
#> Error: Can't modify read-only reactive value 'count'
```
`Chuong 8` se hoc cach su dung ham `updateNumericInput()` de thay doi gia tri trong browser. 
<br>`renderText()` hoac `reactive()` cho phep tu dong cap nhat output khi input thay doi
```
server <- function(input, output, session) {
  message("The value of input$count is ", input$count)
}

shinyApp(ui, server)
#> Error: Can't access reactive value 'count' outside of reactive consumer.
#> ℹ Do you need to wrap inside reactive() or observer()?
```
Mot vi du khac:
```
ui <- fluidPage(
  textOutput("greeting")
)

server <- function(input, output, session) {
  output$greeting <- renderText("Hello human!")
}
```
`render` function co 2 chuc nang: 
- thiet lap mot moi truong phan ung dac biet (special reactive context) de theo doi tu dong input va output
- chuyen doi output trong code R thanh HTML de bieu dien tren trang web
<br> Mot vai loi co the gap:
- quên dùng `render` function
```
server <- function(input, output, session) {
  output$greeting <- "Hello human"
}
shinyApp(ui, server)
#> Error: Unexpected character object for output$greeting
#> ℹ Did you forget to use a render function?
```
- co gang doc mot `output`
```
server <- function(input, output, session) {
  message("The greeting is ", output$greeting)
}
shinyApp(ui, server)
#> Error: Reading from shiny output object is not allowed.
```
Phan biet `imperative programming` va `declarative programming`:
- **imperative programming** command: yêu cầu phải được thực hiện ngay lập tức (command R de load data, hinh anh hoa data,...)
- **declarative programming** dien ta muc tieu cau hon hoac mo ta cac rang buoc quan trong, phu thuoc vao nguoi khac de quyet dinh nhu the nao hoac khi nao chuyen doi thanh hanh dong (Shiny)
## Reactive graph - reactive expression
<img width="140" alt="TEST" src="https://github.com/thiendattran/R_shiny/assets/123424766/5fe6e268-5882-45bd-ab00-657cecc4ea53">

<img width="262" alt="TEST" src="https://github.com/thiendattran/R_shiny/assets/123424766/5b0b6947-be79-4699-88be-136318496a76">

<img width="656" alt="Screenshot 2024-03-31 at 03 21 36" src="https://github.com/thiendattran/R_shiny/assets/123424766/a7a4982c-1ada-49ac-b121-6c7ff756a3d8">

```
server <- function(input, output, session) {
  string <- reactive(paste0("Hello ", input$name, "!"))
  output$greeting <- renderText(string())
}
```
## Thứ tự thực hiện lệnh
```
server <- function(input, output, session) {
  output$greeting <- renderText(string())
  string <- reactive(paste0("Hello ", input$name, "!"))
}
```
<br> **Trong R** code duoc chay theo thu tu tu tren xuong duoi. 
<br> **Trong Shiny** code chi chay khi can. Code chi chay khi `name` thay doi
<br> [chi tiết thực hiện code](https://mastering-shiny.org/reactive-graph.html) 

## [BAI TAP](https://mastering-shiny.org/basic-reactivity.html)
## Reactive expressions (tiep theo)
Đầu tiên tạo 2 hàm `freqpoly()` và `t_tést()`
```
library(ggplot2)

freqpoly <- function(x1, x2, binwidth = 0.1, xlim = c(-3, 3)) {
  df <- data.frame(
    x = c(x1, x2),
    g = c(rep("x1", length(x1)), rep("x2", length(x2)))
  )

  ggplot(df, aes(x, colour = g)) +
    geom_freqpoly(binwidth = binwidth, size = 1) +
    coord_cartesian(xlim = xlim)
}

t_test <- function(x1, x2) {
  test <- t.test(x1, x2)
  
  # use sprintf() to format t.test() results compactly
  sprintf(
    "p value: %0.3f\n[%0.2f, %0.2f]",
    test$p.value, test$conf.int[1], test$conf.int[2]
  )
}
```
Tạo app
```
ui <- fluidPage(
  fluidRow(
    column(4, 
      "Distribution 1",
      numericInput("n1", label = "n", value = 1000, min = 1),
      numericInput("mean1", label = "µ", value = 0, step = 0.1),
      numericInput("sd1", label = "σ", value = 0.5, min = 0.1, step = 0.1)
    ),
    column(4, 
      "Distribution 2",
      numericInput("n2", label = "n", value = 1000, min = 1),
      numericInput("mean2", label = "µ", value = 0, step = 0.1),
      numericInput("sd2", label = "σ", value = 0.5, min = 0.1, step = 0.1)
    ),
    column(4,
      "Frequency polygon",
      numericInput("binwidth", label = "Bin width", value = 0.1, step = 0.1),
      sliderInput("range", label = "range", value = c(-3, 3), min = -5, max = 5)
    )
  ),
  fluidRow(
    column(9, plotOutput("hist")),
    column(3, verbatimTextOutput("ttest"))
  )
)
```
Note:
- fluidRow() đầu tiên: tạo dòng đầu tiên
- fluidRow() thứ hai: tạo dòng thứ 2
- column(): tạo cột
- Số 4 trong column(): Về cơ bản, mỗi dòng sẽ có 12 ô. Số 4 muốn nói lấy 4 ô trong 12 ô tương ứng vs độ rộng của cột
```
server <- function(input, output, session) {
  output$hist <- renderPlot({
    x1 <- rnorm(input$n1, input$mean1, input$sd1)
    x2 <- rnorm(input$n2, input$mean2, input$sd2)
    
    freqpoly(x1, x2, binwidth = input$binwidth, xlim = input$range)
  }, res = 96)

  output$ttest <- renderText({
    x1 <- rnorm(input$n1, input$mean1, input$sd1)
    x2 <- rnorm(input$n2, input$mean2, input$sd2)
    
    t_test(x1, x2)
  })
}
```
## The reactive graph (tiếp theo)
- Shiny đủ thông minh để biết chỉ update khi inpute thay đổi
- Shiny không đủ thông minh để chạy chọn lọc một phần code. Nghĩa là một là chạy tất cả hai là không chạy.
- Xem ví dụ về dưới để thấy tầm quan trọng của reactive expression.
     - 1 phần code đã được dùng ở trên:
```
x1 <- rnorm(input$n1, input$mean1, input$sd1)
x2 <- rnorm(input$n2, input$mean2, input$sd2)
t_test(x1, x2)
```
<img width="229" alt="Screenshot 2024-03-31 at 03 39 22" src="https://github.com/thiendattran/R_shiny/assets/123424766/ad620195-cc0d-4fc5-8532-f2506ac32ef7">

<be> Nếu thay đổi n1, giá trị khác cũng sẽ được update vì Shiny chỉ nhìn vào đầu ra
- Giải pháp: dùng `reactive()`
```
server <- function(input, output, session) {
  x1 <- reactive(rnorm(input$n1, input$mean1, input$sd1))
  x2 <- reactive(rnorm(input$n2, input$mean2, input$sd2))

  output$hist <- renderPlot({
    freqpoly(x1(), x2(), binwidth = input$binwidth, xlim = input$range)
  }, res = 96)

  output$ttest <- renderText({
    t_test(x1(), x2())
  })
}
```
      - reactive graph trở thành
  
<img width="299" alt="Screenshot 2024-03-31 at 03 54 01" src="https://github.com/thiendattran/R_shiny/assets/123424766/0b82c310-da56-4210-8309-61420f5117ec">

## Kiểm soát thời gian
Chúng ta có thể tăng hoặc giảm số lần reactive expression được thực hiện (chapter 15)

## Timed invalidation
`reactiveTimer()` để tăng tần suất update

```
freqpoly <- function(x1, x2, binwidth = 0.1, xlim = c(-3, 3)) {
  df <- data.frame(
    x = c(x1, x2),
    g = c(rep("x1", length(x1)), rep("x2", length(x2)))
  )
  
  ggplot(df, aes(x, colour = g)) +
    geom_freqpoly(binwidth = binwidth, size = 1) +
    coord_cartesian(xlim = xlim)
}

ui <- fluidPage(
  fluidRow(
    column(3, 
      numericInput("lambda1", label = "lambda1", value = 3),
      numericInput("lambda2", label = "lambda2", value = 5),
      numericInput("n", label = "n", value = 1e4, min = 0)
    ),
    column(9, plotOutput("hist"))
  )
)
server <- function(input, output, session) {
  x1 <- reactive(rpois(input$n, input$lambda1))
  x2 <- reactive(rpois(input$n, input$lambda2))
  output$hist <- renderPlot({
    freqpoly(x1(), x2(), binwidth = 1, xlim = c(0, 40))
  }, res = 96)
}

server <- function(input, output, session) {
  timer <- reactiveTimer(500)
  
  x1 <- reactive({
    timer()
    rpois(input$n, input$lambda1)
  })
  x2 <- reactive({
    timer()
    rpois(input$n, input$lambda2)
  })
  
  output$hist <- renderPlot({
    freqpoly(x1(), x2(), binwidth = 1, xlim = c(0, 40))
  }, res = 96)
}
```
<img width="677" alt="Screenshot 2024-03-31 at 04 11 32" src="https://github.com/thiendattran/R_shiny/assets/123424766/e524ef0a-48cc-4253-be80-900d3c0f9e48">

## Nhấp chuột
Trong tình huống ở trên, nếu mô phỏng mất 1s để chạy xong mà mô phỏng đc cập nhật mỗi 0.5s sẽ dẫn tới việc Shiny sẽ phải làm nhiều để đuổi kịp. Hoặc nếu ai đó nhấp chuột nhanh vào các nút trong ứng dụng trong khi việc tính toán đang thực hiện tương đối tốn thời gian sẽ tạo ra một số lượng lón công việc chờ Shiny và khi nó đang làm việc trong một khối lượng lớn công việc chờ như vậy nó sẽ không thể phản ứng vs bất kì sự việc mới. Hậu quả là trải nghiệm ng dùng kém.
<br>
<br> Giải pháp: dùng `actionButtion()`
```
freqpoly <- function(x1, x2, binwidth = 0.1, xlim = c(-3, 3)) {
  df <- data.frame(
    x = c(x1, x2),
    g = c(rep("x1", length(x1)), rep("x2", length(x2)))
  )
  
  ggplot(df, aes(x, colour = g)) +
    geom_freqpoly(binwidth = binwidth, size = 1) +
    coord_cartesian(xlim = xlim)
}

ui <- fluidPage(
  fluidRow(
    column(3, 
      numericInput("lambda1", label = "lambda1", value = 3),
      numericInput("lambda2", label = "lambda2", value = 5),
      numericInput("n", label = "n", value = 1e4, min = 0),
      actionButton("simulate", "Simulate!")
    ),
    column(9, plotOutput("hist"))
  )
)

server <- function(input, output, session) {
  x1 <- reactive({
    input$simulate
    rpois(input$n, input$lambda1)
  })
  x2 <- reactive({
    input$simulate
    rpois(input$n, input$lambda2)
  })
  output$hist <- renderPlot({
    freqpoly(x1(), x2(), binwidth = 1, xlim = c(0, 40))
  }, res = 96)
}
```
<img width="684" alt="Screenshot 2024-03-31 at 04 29 34" src="https://github.com/thiendattran/R_shiny/assets/123424766/1ca49c93-8319-48ff-ab05-20e34925b095">

Vấn đề: graph sẽ được update khi bấm vào nút `simulate` nhưng nó cũng sẽ được update khi thay đổi n, lamda1,..
<br> Giải pháp: `eventReactive()` gồm 2 tham số: phụ thuộc vào cái igf và tính toán cái gì. Nghiã là nó sẽ chỉ tính x1() và x2() khi `simulate` được click. 

````
server <- function(input, output, session) {
  x1 <- eventReactive(input$simulate, {
    rpois(input$n, input$lambda1)
  })
  x2 <- eventReactive(input$simulate, {
    rpois(input$n, input$lambda2)
  })

  output$hist <- renderPlot({
    freqpoly(x1(), x2(), binwidth = 1, xlim = c(0, 40))
  }, res = 96)
}
```

<img width="680" alt="Screenshot 2024-03-31 at 04 39 52" src="https://github.com/thiendattran/R_shiny/assets/123424766/f5f21e6b-02c8-435c-8c12-b6f1792b1df6">

## Observers
Công cụ thông báo observeEvent() console sẽ nhận đc thông báo mỗi khi 'name' được cập nhật
```
ui <- fluidPage(
  textInput("name", "What's your name?"),
  textOutput("greeting")
)

server <- function(input, output, session) {
  string <- reactive(paste0("Hello ", input$name, "!"))
  
  output$greeting <- renderText(string())
  observeEvent(input$name, {
    message("Greeting performed")
  })
}
```








