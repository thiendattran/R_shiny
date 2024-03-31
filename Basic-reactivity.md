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
<br>Shiny đủ thông minh để biết chỉ update khi inpute thay đổi
<br>Shiny không đủ thông minh để chạy chọn lọc một phần code. Nghĩa là một là chạy tất cả hai là không chạy.
<br>Xem ví dụ về dưới để thấy tầm quan trọng của reactive expression. 

<img width="229" alt="Screenshot 2024-03-31 at 03 39 22" src="https://github.com/thiendattran/R_shiny/assets/123424766/ad620195-cc0d-4fc5-8532-f2506ac32ef7">




