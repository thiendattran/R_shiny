# Tự học R - Shiny

## Tài liệu
<img width="257" alt="image" src="https://github.com/thiendattran/R_shiny/assets/120559860/78bb8f88-5717-4d3a-89c1-7989b940fe3f">

[Mastering Shiny](https://mastering-shiny.org/?fbclid=IwAR1PEqbhGtWVgtWBvjdZ9VlcI8MI0z7n5V63msuSeXM5BqafMk9focTB2_U
)

## Lịch học
- [Link webex](https://meduniwien.webex.com/meet/thien.tran)
- [Google Sheet đăng ký bài](https://docs.google.com/spreadsheets/d/16Qrs1u7-ykc-9ZqkH2024pqRl42_-Qk5bZw1ss_SIiQ/edit?usp=sharing)

| STT | Ngày | Section | Lead |
| --- | --- | --- | --- |
|1|24/03|Cơ bản về Rshiny| Đạt|
|2|31/03|...|...|

## Tổng quan
Shiny là R package được dùng để tạo ra web app tương tác được (interactive web app).

Thay vì cần các kiến thức về web coding như HTML, CSS hay JavaScript.

Shiny làm việc này cho người học code một cách dễ dàng hơn hoàn toàn từ R.

Các ứng dụng của R Shiny
- Tạo Dashboards
- Thay vì giới thiệu vài trang pdf thì chỉ cần sử dụng tương tác để xem chính xác kết quả cần xem.
- Dễ hiểu cho người xem không chuyên môn
- Tạo tool cho người khác tương tác, gửi email và upload data của chính mình lên phân tích
- Tạo ví dụ trong quá trình dạy statistics hoặc modelling

 
## Các package cần sử dụng trong sách này
Code để cài các package dùng trong sách này
```
install.packages(c(
  "gapminder", "ggforce", "gh", "globals", "openintro", "profvis", 
  "RSQLite", "shiny", "shinycssloaders", "shinyFeedback", 
  "shinythemes", "testthat", "thematic", "tidyverse", "vroom", 
  "waiter", "xml2", "zeallot" 
))
```

## Basic in shiny
### Ví du cơ bản
Một app.R Shiny đơn giản nhất có thể như thế này
```
library(shiny)
ui <- fluidPage(
  "Hello, world!"
)
server <- function(input, output, session) {
}
shinyApp(ui, server)
```
Với bốn thành phần bao gồm:
- Gọi library(shiny)
- Tạo User Interface (UI) - Hello world
- Tạo server
- Chạy shinyApp(ui,server) để chạy application

Lưu ý: Khi R app đang chạy, R shiny sẽ tạm thời block R console, không thể sử dụng console khi R Shiny app đang chạy.

### Các UI control options
#### Tạo bảng với nhiều lựa chọn 
```
ui <- fluidPage(
  selectInput("dataset", label = "Dataset", choices = ls("package:datasets")),
  verbatimTextOutput("summary"),
  tableOutput("table")
)
```
Sử dụng các function:
- fluidPage(): tạo layout function. Xem thêm từ chapter 6
- selectInput(): tạo **input control**, dùng cho user có thể chọn lựa chọn trong **choices**
- verbatimTextOutput() và tableOutput: để render kết  sau khi user chọn xong input

### Thêm behaviour
Tạo behaviour cho server
```
server <- function(input, output, session) {
  output$summary <- renderPrint({
    dataset <- get(input$dataset, "package:datasets")
    summary(dataset)
  })
  
  output$table <- renderTable({
    dataset <- get(input$dataset, "package:datasets")
    dataset
  })
}
```
Để Shiny app có tính tương tác (interactive). Có thẻ dùng server để chạy các function tính toán và render kết quả.

Các function sẽ được viết dưới dạng `render{Type}`, ví dụ `renderPrint` hay `renderTable` tạo ra các text  hoặc table để gán vào output ở dạng `output${Type}`.

`renderPrint` sẽ được gắn vào `verbatimTextOutput` ,`renderTable` sẽ được gắn vào `tableOutput` để hiện thị lên UI

## Reactive - Hạn chế các dòng code bị duplicate
Ở ví dụ trên dòng dataset phải duplicate 2 lần để chạy trong output summary và output table
```
dataset <- get(input$dataset, "package:datasets")
```
Để xử lý trường hợp này trong Rshiny, ta sử dụng **reactive expression**. Dùng `reactive({..})` để bao quanh code bị duplicate và gán vào variable.

```
server <- function(input, output, session) {
  # Create a reactive expression
  dataset <- reactive({
    get(input$dataset, "package:datasets")
  })

  output$summary <- renderPrint({
    # Use a reactive expression by calling it like a function
    summary(dataset())
  })
  
  output$table <- renderTable({
    dataset()
  })
}
```
Trong trường hợp này dataset chỉ cần 1 lần retrieve từ input. Shiny app sẽ gọn và chạy nhanh hơn.

## Cheatsheet
[Link cheatsheet](https://rstudio.github.io/cheatsheets/shiny.pdf)

## Contents
- [Basic UI](básicui.md)
- [Basic Reactive]





