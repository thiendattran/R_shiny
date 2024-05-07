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

**Progress bars**
<br>Tao thanh tien trinh vs ```withProgress()``` va ```incProgress()```
- ```withProgress()```: xuat hien khi bat dau chay code va bien mat khi code da chay xong
- ```incProgress()```: su tang dan cua thanh $\color{blue}{PROGRESS}$
```
ui <- fluidPage(
  numericInput("steps", "How many steps?", 10),
  actionButton("go", "go"),
  textOutput("result")
)

server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    withProgress(message = "Computing random number", {
      for (i in seq_len(input$steps)) {
        Sys.sleep(0.5)
        incProgress(1 / input$steps)
      }
      runif(1)
    })
  })
  
  output$result <- renderText(round(data(), 2))
}
shinyApp(ui = ui, server = server)
```
![image](https://github.com/thiendattran/R_shiny/assets/123424766/5a758eee-f954-42aa-9f14-6855c92c8352)

```
library(waiter)
ui <- fluidPage(
  waiter::use_waitress(),
  numericInput("steps", "How many steps?", 10),
  actionButton("go", "go"),
  textOutput("result")
)

server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    waitress <- waiter::Waitress$new(max = input$steps)
    on.exit(waitress$close())
    
    for (i in seq_len(input$steps)) {
      Sys.sleep(0.5)
      waitress$inc(1)
    }
    
    runif(1)
  })
  
  output$result <- renderText(round(data(), 2))
}
shinyApp(ui = ui, server = server)
```
```waiter::use_waiter()```: su dung tinh nang man hinh tai
<br>
```eventReactive()```: kich hoat block of code ben trong khi user click vao nut $\color{blue}{go}$
<br>
```on.exit(waitress$close())```: dam bao rang $\color{green}{progress \ bar}$ se bien mat khi code chay xong hoac xay ra loi trong qua trinh chay code
<br>
```waitress$inc(1)```: su tang dan cua thanh $\color{green}{progress \ bar}$ moi 1 unit
<br>
```runif(1)```: chay 1 so ngau nhien tu 0 den 1 khi vong lap hoan thanh
<br>

**Progress bar de len phan input**
- overlay: progress bar mo duc an toan bo input
- overlay-opacity: progress bar mo duc phu len input
- overlay-percent: progress bar mo duc dong thoi hien so %

```
ui <- fluidPage(
  waiter::use_waitress(),
  numericInput("steps", "How many steps?", 10),
  actionButton("go", "go"),
  textOutput("result")
)

server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    waitress <- Waitress$new(selector = "#steps", theme = "overlay-percent")
    on.exit(waitress$close())
    
    for (i in seq_len(input$steps)) {
      Sys.sleep(0.5)
      waitress$inc(1)
    }
    
    runif(1)
  })
  
  output$result <- renderText(round(data(), 2))
}
shinyApp(ui = ui, server = server)
```
**Spinners (VONG QUAY)**
```
ui <- fluidPage(
  waiter::use_waiter(),
  actionButton("go", "go"),
  textOutput("result")
)

server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    waiter <- waiter::Waiter$new()
    waiter$show()
    on.exit(waiter$hide())
    
    Sys.sleep(sample(5, 1))
    runif(1)
  })
  output$result <- renderText(round(data(), 2))
}
shinyApp(ui = ui, server = server)
```
Them **id=** de spinner khong che phu trang

```
ui <- fluidPage(
  waiter::use_waiter(),
  actionButton("go", "go"),
  textOutput("result")
)

server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    waiter <- waiter::Waiter$new(id= "result")
    waiter$show()
    on.exit(waiter$hide())
    
    Sys.sleep(sample(5, 1))
    runif(1)
  })
  output$result <- renderText(round(data(), 2))
}
shinyApp(ui = ui, server = server)
```

```
ui <- fluidPage(
  waiter::use_waiter(),
  actionButton("go", "go"),
  textOutput("result")
)

server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    waiter <- waiter::Waiter$new(id= "result")$show()
    on.exit(waiter$hide())
    
    Sys.sleep(sample(5, 1))
    runif(1)
  })
  output$result <- renderText(round(data(), 2))
}
shinyApp(ui = ui, server = server)
```

```
ui <- fluidPage(
  waiter::use_waiter(),
  actionButton("go", "go"),
  plotOutput("plot"),
)

server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    waiter::Waiter$new(html = spin_ripple())$show()
    
    Sys.sleep(2)
    data.frame(x = runif(50), y = runif(50))
  })
  
  output$plot <- renderPlot(plot(data()), res = 96)
}
shinyApp(ui = ui, server = server)
```
Mot kieu khac voi ```shinycssloaders::withSpinner()```

```
library(shinycssloaders)

ui <- fluidPage(
  actionButton("go", "go"),
  withSpinner(plotOutput("plot")),
)
server <- function(input, output, session) {
  data <- eventReactive(input$go, {
    Sys.sleep(3)
    data.frame(x = runif(50), y = runif(50))
  })
  
  output$plot <- renderPlot(plot(data()), res = 96)
}
shinyApp(ui = ui, server = server)
```
**XAC NHAN VA HUY BO**

```
library(shinyjs)
ui <- fluidPage(
  useShinyjs(), 
  fileInput('file', 'Data', buttonLabel ="Upload"),
  actionButton("delete","Delete file")
)

server <- function(input, output, session) {
  
    modal_confirm <- modalDialog(
  "Are you sure you want to delete?",
  title = "Deleting files",
  footer = tagList(
    actionButton("cancel", "Cancel"),
    actionButton("ok", "Delete", class = "btn btn-danger")
  )
)
    
  observeEvent(input$delete, {
    if(!is.null(input$file)){
       showModal(modal_confirm)
    } else {
      showNotification("No file uploaded", type = "message")
    }
   
  })
  
  observeEvent(input$ok, {
    removeModal()
     showNotification("Files deleted", type ="message")
     reset("file")
  })
  observeEvent(input$cancel, {
    removeModal()
  })
}
shinyApp(ui = ui, server = server)
```

![image](https://github.com/thiendattran/R_shiny/assets/123424766/a5eaaef2-8def-4eb9-91e5-244a1298d785)

$\color{red}{LUU \ Y:}$
- shiny khong ho tro xoa input. Neu muon xoa file input, su dung ```shinyjs::useShinyjs()``` va ```reset()```
- tao **dialog box** voi ```modalDialog()```
- showModal(): hien hop thoai
- removeModal(): an hop thoai






































