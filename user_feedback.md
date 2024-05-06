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

![Capture](https://github.com/thiendattran/R_shiny/assets/123424766/66f2d319-876b-4dc7-b666-7391c841b2e1)

```
ui <- fluidPage(
  selectInput("language", "Language", choices = c("", "English", "French","Vietnamese")),
  textInput("name", "Name"),
  textOutput("greeting")
)

server <- function(input, output, session) {
  greetings <- c(
    English = "Hello", 
    French = "Bonjour",
    Vietnamese = "Xin chao"
  )
  output$greeting <- renderText({
    paste0(greetings[[input$language]], " ", input$name, "!")
  })
}
shinyApp(ui= ui, server=server)
```
![Capture](https://github.com/thiendattran/R_shiny/assets/123424766/1f7bdfb7-8f48-4047-add6-d1e1c73e5d89)

```
ui <- fluidPage(
  selectInput("language", "Language", choices = c("", "English", "French","Vietnamese")),
  textInput("name", "Name"),
  textOutput("greeting")
)

server <- function(input, output, session) {
  greetings <- c(
    English = "Hello", 
    French = "Bonjour",
    Vietnamese = "Xin chao"
  )
  output$greeting <- renderText({
    req(input$language, input$name)
    paste0(greetings[[input$language]], " ", input$name, "!")
  })
}
shinyApp(ui= ui, server=server)
```

```
ui <- fluidPage(
  shinyFeedback::useShinyFeedback(),
  textInput("dataset", "Dataset name"), 
  tableOutput("data")
)

server <- function(input, output, session) {
  data <- reactive({
    req(input$dataset)
    
    exists <- exists(input$dataset, "package:datasets")
    shinyFeedback::feedbackDanger("dataset", !exists, "Unknown dataset")
    req(exists, cancelOutput = TRUE)

    get(input$dataset, "package:datasets")
  })
  
  output$data <- renderTable({
    head(data())
  })
}
shinyApp(ui= ui, server=server)

```
$\color{red}{Nhuoc \ diem \ cua \ shinyFeedback:}$
<br>Khi co nhieu input thi message ```error``` se hien thi o input nao???
<br> **validate()**
```
ui <- fluidPage(
  numericInput("x", "x", value = 0),
  selectInput("trans", "transformation", 
    choices = c("square", "log", "square-root")
  ),
  textOutput("out")
)

server <- function(input, output, session) {
  output$out <- renderText({
    if (input$x < 0 && input$trans %in% c("log", "square-root")) {
      validate("x can not be negative for this transformation")
    }
    
    switch(input$trans,
      square = input$x ^ 2,
      "square-root" = sqrt(input$x),
      log = log(input$x)
    )
  })
}
```
$\color{red}{Hien \ thi \ thong \ bao \ voi \showNotification():}$
```
ui <- fluidPage(
  actionButton("goodnight", "Good night")
)
server <- function(input, output, session) {
  observeEvent(input$goodnight, {
    showNotification("So long")
    Sys.sleep(1)
    showNotification("Farewell")
    Sys.sleep(1)
    showNotification("Auf Wiedersehen")
    Sys.sleep(1)
    showNotification("Adieu")
  })
}
shinyApp(ui= ui, server=server)
```
$\color{red}{LUU \ Y:}$
Doan tin nhan se xuat hien trong 5s
Them ```color``` de phan biet kieu tin nhan voi ```type=" "```

```
ui <- fluidPage(
  actionButton("goodnight", "Good night")
)
server <- function(input, output, session) {
  observeEvent(input$goodnight, {
    showNotification("So long")
    Sys.sleep(1)
    showNotification("Farewell", type = "message")
    Sys.sleep(1)
    showNotification("Auf Wiedersehen", type = "warning")
    Sys.sleep(1)
    showNotification("Adieu", type = "error")
  })
}
shinyApp(ui= ui, server=server)
```
![image](https://github.com/thiendattran/R_shiny/assets/123424766/d76963c9-6b88-45ca-9967-1fd388da22e0)


$\color{blue}{Hien \ thi \ tin \ nhan}$ va $\color{blue}{tu \ dong \ bien \ mat}$ sau khi cong viec duoc hoan thanh voi ```showNotification()``` va ```on.exist(removeNotification())```
```
ui <- fluidPage(
  actionButton("startTask", "Start Task"),
  dataTableOutput("data_table")
)

server <- function(input, output, session) {

  observeEvent(input$startTask, {

    notification <- showNotification("Reading data...", duration = NULL, closeButton = FALSE)
    
    Sys.sleep(3)
    data <- iris

    on.exit(removeNotification(notification))

    output$data_table <- renderDataTable({
      data
    })
  })
}

shinyApp(ui = ui, server = server)
```

**Progressive updates**
```
ui <- fluidPage(
  tableOutput("data")
)

server <- function(input, output, session) {
  notify <- function(msg, id = NULL) {
    showNotification(msg, id = id, duration = NULL, closeButton = FALSE)
  }

  data <- reactive({ 
    id <- notify("Reading data...")
    on.exit(removeNotification(id), add = TRUE)
    Sys.sleep(2)
      
    notify("Reticulating splines...", id = id)
    Sys.sleep(2)
    
    notify("Herding llamas...", id = id)
    Sys.sleep(2)

    notify("Orthogonalizing matrices...", id = id)
    Sys.sleep(2)
        
    mtcars
  })
  
  output$data <- renderTable(head(data()))
}
shinyApp(ui = ui, server = server)
```













































