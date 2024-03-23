## 2 Basic UI

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

