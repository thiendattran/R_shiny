# 2 Basic UI

Trong chapter này chúng ta sẽ tập trung vào front end, vài ví dụ giới thiệu về HTML input và output

### 2.2 Inputs
Các function cơ bản trong input sẽ bao gồm
- sliderInput()
- selectInput()
- textInput()
- numericInput()

### Cấu trúc căn bản trong input
Trong các functinon input, thì argument đầu tiên sẽ là `inputId`. Đây là dòng code để kết nối frontend với backend, nếu Inputid là `name` thì server sẽ có dữ liệu ở `input$name`.

Trong `inputId` có 2 đặc điểm:
- Phải là string chỉ bao gồn chữ, số và _ (không chứa space, dash, period tương tự như cách đặt tên biếng trong R)
- Phải unique

Argument thứ 2 trong các input function là `label`, dùng để hiển thị nội dung để hướng dẫn user nhập dữ liệu vào

Ví dụ tạo slider:
```
sliderInput("min", "Limit (minimum)", value = 50, min = 0, max = 100)
```
Trong ví dụ này input function `sliderInput` có `inputId` là min, sẽ tạo ra biến input$min trong server và `label` là "Limit (minimum)" để hiện thị trên UI.

### Free text (dùng cho user nhập văn bản)
Sử dụng function `textInput()` hoặc `textAreaInput()`(tạo ô nhập văn bản rộng hơn)

```
ui <- fluidPage(
  textInput("name", "What's your name?"),
  passwordInput("password", "What's your password?"),
  textAreaInput("story", "Tell me about yourself", rows = 3)
)
```

### Numeric inputs
Để thu thập dữ liệu dạng số có thể dùng function `numericInput()` hoặc slider với `sliderInput()`. slider còn có thể dùng làm range

```
ui <- fluidPage(
  numericInput("num", "Number one", value = 0, min = 0, max = 100),
  sliderInput("num2", "Number two", value = 50, min = 0, max = 100),
  sliderInput("rng", "Range", value = c(10, 20), min = 0, max = 100)
)
```

Slider chỉ nên dùng với các giá trị không cần độ chính xác cao.

### Dates
Tạo calendar cho user chọn ngày với `dateInput()` hoặc `dateRangeInput()` nếu muốn chọn nhiều ngày, ngoài ra có thể sử dụng `datesdisabled` và `daysofweekdisabled` để giới hạn ngày được chọn cho user
```
ui <- fluidPage(
  dateInput("dob", "When were you born?"),
  dateRangeInput("holiday", "When do you want to go on vacation next?")
)
```

### Limited choices
Trước tiên tạo một list các option để user chọn, sau đó đưa vào `selectInput()` hoặc `radioButtons` để tạo nút chọn (radio button chỉ chọn được một đáp án
```
animals <- c("dog", "cat", "mouse", "bird", "other", "I hate animals")

ui <- fluidPage(
  selectInput("state", "What's your favourite state?", state.name),
  radioButtons("animal", "What's your favourite animal?", animals)
)
```
Radio button còn có thể thay đổi giữa hiển thị lựa chọn và code của variablé, ví dụ sử dụng emoji cho các biến cảm xúc
```
ui <- fluidPage(
  radioButtons("rb", "Choose one:",
    choiceNames = list(
      icon("angry"),
      icon("smile"),
      icon("sad-tear")
    ),
    choiceValues = list("angry", "happy", "sad")
  )
)
```
### Dropdown
Tương tự như Radio button nhưng có thể sử dụng trong các trường hợp list quá dài
```
ui <- fluidPage(
  selectInput(
    "state", "What's your favourite state?", state.name,
    multiple = TRUE
  )
)
```
### Trường hợp user cần chọn nhiều phương án
Sử dụng checkbox nếu muốn user chọn nhiều phương án
```
ui <- fluidPage(
  checkboxGroupInput("animal", "What animals do you like?", animals)
)
```

### Tạo nơi upload file cho user
Sử dụng `fileInput()`

```
ui <- fluidPage(
  fileInput("upload", NULL)
)
```

### Action button
Sử dụng action button để kết nối với `observeEvent()` hoặc `eventReactive()
```
ui <- fluidPage(
  fluidRow(
    actionButton("click", "Click me!", class = "btn-danger"),
    actionButton("drink", "Drink me!", class = "btn-lg btn-success")
  ),
  fluidRow(
    actionButton("eat", "Eat me!", class = "btn-block")
  )
)
```

## Output
Output trong UI tạo ra bằng cách viết "placeholder" giữ chổ để server chạy function ra kết quả. Nếu UI giữ chỗ trước bnagwf cách tạo output với ID là "plot" thì trong server sẽ access bằng `output$plot`.

Mỗi output function ở frontend sẽ được dính với chữ `render` ở trước, shiny có 3 dạng dính cho output là text, table, plot.

### Text
Output ở dạng text sẽ được gọi bằng `textOutput()` hoặc `verbatimTextoutput()` để xuất ra output ở console khi chạy 
```
ui <- fluidPage(
  textOutput("text"),
  verbatimTextOutput("code")
)
server <- function(input, output, session) {
  output$text <- renderText({ 
    "Hello friend!" 
  })
  output$code <- renderPrint({ 
    summary(1:10) 
  })
}
``` 
Chú ý là trong `render` function, dấu ngoặc {} chỉ cần thiết khi render phải sử dụng nhiều dòng code. Thường thì trong shiny không nên viết quá nhiều dòng trong function `render` nên code thường thấy sẽ ở dạng
```
server <- function(input, output, session) {
  output$text <- renderText("Hello friend!")
  output$code <- renderPrint(summary(1:10))
}
```
- `renderText()` được dùng để xuất ra 1 string được dùng cặp với `textOutput`
- `renderPrint()` được dùng để print kết quả, như là kết quả của console R, dùng cặp với `verbatimTextoutput`
Khác biệt:
```
ui <- fluidPage(
  textOutput("text"),
  verbatimTextOutput("print")
)
server <- function(input, output, session) {
  output$text <- renderText("hello!")
  output$print <- renderPrint("hello!")
}
```
### Tables
Có 2 lựa chọn khác nhau khi hiện thị dataframe dạng table:
- `tableOutput()` và `renderTable()` dùng để show hết mới data cùng một lúc
- `dataTableOutput()` và `renderDataTable()` để tạo bảng dynamic, chỉ show đúng số dòng với số lượng dòng nhất định.

`tableOutput` chỉ nên sử dụng với những output nhỏ và cố định như summaries, còn trong trường hợp muốn xuất ra dataframe nên sử dụng `dataTableOutput()`


```
ui <- fluidPage(
  tableOutput("static"),
  dataTableOutput("dynamic")
)
server <- function(input, output, session) {
  output$static <- renderTable(head(mtcars))
  output$dynamic <- renderDataTable(mtcars, options = list(pageLength = 5))
}
```


### Plot
Shiny có thể dùng để show mọi loại R graphics với `plotOutput()` và `renderPlot()`
```
ui <- fluidPage(
  plotOutput("plot", width = "400px")
)
server <- function(input, output, session) {
  output$plot <- renderPlot(plot(1:5), res = 96)
}
```
Mặc định `plotOutput()` sẽ sử dụng toàn chiều rộng của container = 400 pixels. Ta có thể sử dụng height and width argument để điều, độ phân giải nên được giữ ở `res = 96`.

Plots là dạng output đặc biệt vì có thể làm input trong Shiny. Đọc tiếp ở chapter 7.




