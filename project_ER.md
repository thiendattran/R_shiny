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
- `body_pảt` là phần cơ thể bị tai nạn
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
