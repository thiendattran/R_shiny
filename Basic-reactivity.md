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
- quen dung `render` fucntion
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
#> Error: Reading from shinyoutput object is not allowed.
```
Phan biet `imperative programming` va `declarative programming`:
- **imparative programming** command can duoc thuc hien ngay lap tuc (command R de load data, hinh anh hoa data,...)
- **declarative programming** dien ta muc tieu cau hon hoac mo ta cac rang buoc quan trong, phu thuoc vao nguoi khac de quyet dinh nhu the nao hoac khi nao chuyen doi thanh hanh dong (Shiny)
## Reactive graph 
<img width="140" alt="TEST" src="https://github.com/thiendattran/R_shiny/assets/123424766/5fe6e268-5882-45bd-ab00-657cecc4ea53">

<br> **Trong R** code duoc chay theo thu tu tu tren xuong duoi. 
<br> **Trong Shiny** code chi chay khi can. Code chi chay khi `name` thay doi

## Reactive expressions
<img width="262" alt="TEST" src="https://github.com/thiendattran/R_shiny/assets/123424766/5b0b6947-be79-4699-88be-136318496a76">

Cong cu de giam su lap lai trong reactive code
## Thu tu chay code
```
server <- function(input, output, session) {
  output$greeting <- renderText(string())
  string <- reactive(paste0("Hello ", input$name, "!"))
}
```
$\color{green}{Thu \ tu \ code \ khong \ la \ mot \ loi \ trong \ Shiny}$. Boi vi Shiny rat luoi nen code chi chay khi qua trinh duoc kich hoat, nghia la sau khi `string` duoc tao ra

## [BAI TAP](https://mastering-shiny.org/basic-reactivity.html)
## Reactive expressions (tiep theo)
<img width="297" alt="TEST" src="https://github.com/thiendattran/R_shiny/assets/123424766/1109e9f6-4552-42b6-82ba-f5876d2c7833">

Chung ta se su dung mot app phuc tap hon de thay duoc loi ich cua reactive expressions. 



