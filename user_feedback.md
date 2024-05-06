# USER FEEDBACK

**library(shiny)**

```
ui <- fluidPage(
  shinyFeedback::useShinyFeedback(),
  numericInput("n", "n", value = 10),
  textOutput("half")
)

server <- function(input, output, session) {
  half <- reactive({
    even <- input$n %% 2 == 0            
    shinyFeedback::feedbackWarning("n", !even, "Please select an even number")
    input$n / 2    
  })
  
  output$half <- renderText(half())
}

shinyApp(ui= ui, server=server)
```
$\color{red}{LUU \ Y:}$
<br>**n %% 2 ==0: phan du cua phep chia cho 2 la 0**


![Capture](https://github.com/thiendattran/R_shiny/assets/123424766/35b796bb-a370-4b8d-9ee9-79577b3990f5)


No van hien ra ket qua du da bao loi . Su dung ```req()``` de khong hien ra ket qua khi bao loi 
```
ui <- fluidPage(
  shinyFeedback::useShinyFeedback(),
  numericInput("n", "n", value = 10),
  textOutput("half")
)

server <- function(input, output, session) {
  half <- reactive({
    even <- input$n %% 2 == 0
    shinyFeedback::feedbackWarning("n", !even, "Please select an even number")
    req(even)
    input$n / 2    
  })
  
  output$half <- renderText(half())
}

shinyApp(ui= ui, server=server)
```


































