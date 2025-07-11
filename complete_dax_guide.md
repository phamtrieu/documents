# Hướng Dẫn Hoàn Chỉnh DAX Trong Power BI

## Mục Lục
1. [Giới Thiệu Về DAX](#giới-thiệu-về-dax)
2. [Khái Niệm Cơ Bản](#khái-niệm-cơ-bản)
3. [Các Hàm DAX Theo Danh Mục](#các-hàm-dax-theo-danh-mục)
4. [Hàm Nâng Cao và Chuyên Sâu](#hàm-nâng-cao-và-chuyên-sâu)
5. [Context và Filter Context](#context-và-filter-context)
6. [Patterns và Best Practices](#patterns-và-best-practices)
7. [Tình Huống Thực Tế](#tình-huống-thực-tế)
8. [Debugging và Tối Ưu Hóa](#debugging-và-tối-ưu-hóa)
9. [Tài Nguyên Hỗ Trợ](#tài-nguyên-hỗ-trợ)

---

## Giới Thiệu Về DAX

DAX (Data Analysis Expressions) là ngôn ngữ biểu thức mạnh mẽ được thiết kế đặc biệt cho việc phân tích dữ liệu trong Power BI, Power Pivot, và Analysis Services. DAX cho phép bạn tạo ra:

- **Calculated Columns**: Cột tính toán được thêm vào bảng
- **Measures**: Phép tính động được sử dụng trong visualizations  
- **Calculated Tables**: Bảng mới được tạo từ dữ liệu hiện có
- **Row-level Security (RLS)**: Bảo mật cấp hàng

### Đặc Điểm Chính của DAX
- **Functional Language**: Sử dụng các hàm để trả về kết quả
- **Context-Aware**: Kết quả thay đổi theo ngữ cảnh của dữ liệu
- **Column-Based**: Hoạt động trên toàn bộ cột thay vì từng ô
- **Dynamic**: Tự động cập nhật khi dữ liệu thay đổi

---

## Khái Niệm Cơ Bản

### Context trong DAX

#### Row Context
- Ngữ cảnh tại mỗi hàng cụ thể
- Tự động có sẵn trong Calculated Columns
- Cần được tạo ra trong Measures bằng iterator functions

#### Filter Context  
- Tập hợp các bộ lọc được áp dụng lên dữ liệu
- Được tạo bởi visuals, slicers, và các hàm filter
- Ảnh hưởng đến kết quả của Measures

### Evaluation Context
```dax
// Ví dụ về Row Context trong Calculated Column
Profit = Sales[Revenue] - Sales[Cost]

// Ví dụ về Filter Context trong Measure  
Total Sales = SUM(Sales[Revenue]) // Bị ảnh hưởng bởi filters
```

### Data Types trong DAX
- **Whole Number**: Số nguyên 64-bit
- **Decimal Number**: Số thập phân 64-bit
- **Currency**: Tiền tệ với 2 chữ số thập phân
- **Date/Time**: Ngày giờ
- **Text**: Chuỗi ký tự Unicode
- **True/False**: Boolean
- **Binary**: Dữ liệu nhị phân

---

## Các Hàm DAX Theo Danh Mục

### 1. Hàm Tổng Hợp (Aggregation Functions)

#### Hàm Tổng Hợp Cơ Bản

**SUM / SUMX**
```dax
// SUM: Tính tổng một cột
Total Revenue = SUM(Sales[Revenue])

// SUMX: Iterator function, tính tổng với biểu thức
Total Profit = SUMX(Sales, Sales[Revenue] - Sales[Cost])
```

**AVERAGE / AVERAGEX**
```dax
// AVERAGE: Trung bình cột
Avg Order Value = AVERAGE(Sales[OrderAmount])

// AVERAGEX: Trung bình với biểu thức
Avg Profit Margin = AVERAGEX(Sales, DIVIDE(Sales[Profit], Sales[Revenue]))
```

**MIN / MAX / MINX / MAXX**
```dax
// Giá trị min/max của cột
Min Price = MIN(Products[Price])
Max Price = MAX(Products[Price])

// Min/Max với biểu thức
Max Profit = MAXX(Sales, Sales[Revenue] - Sales[Cost])
```

**COUNT / COUNTA / COUNTX / COUNTBLANK**
```dax
// COUNT: Đếm số không rỗng (chỉ số)
Orders Count = COUNT(Sales[OrderID])

// COUNTA: Đếm số không rỗng (tất cả types)
Non Empty Cells = COUNTA(Sales[CustomerName])

// COUNTX: Đếm với điều kiện
High Value Orders = COUNTX(Sales, IF(Sales[Revenue] > 1000, 1, BLANK()))

// COUNTBLANK: Đếm ô trống
Empty Cells = COUNTBLANK(Sales[Comments])
```

#### Hàm Tổng Hợp Nâng Cao

**DISTINCTCOUNT / DISTINCTCOUNTNOBLANK**
```dax
// Đếm giá trị duy nhất
Unique Customers = DISTINCTCOUNT(Sales[CustomerID])
Unique Products = DISTINCTCOUNTNOBLANK(Sales[ProductID])
```

**MEDIAN / PERCENTILE.INC / PERCENTILE.EXC**
```dax
// Trung vị
Median Revenue = MEDIAN(Sales[Revenue])

// Percentile (bao gồm/loại trừ endpoints)
P90_Revenue = PERCENTILE.INC(Sales[Revenue], 0.9)
P95_Revenue = PERCENTILE.EXC(Sales[Revenue], 0.95)
```

**RANKX**
```dax
// Xếp hạng
Product Rank = RANKX(ALL(Products), [Total Sales],, DESC)
Customer Rank = RANKX(ALL(Customers[CustomerID]), [Customer Revenue],, DESC, DENSE)
```

### 2. Hàm Lọc (Filter Functions)

#### Hàm Lọc Cơ Bản

**FILTER**
```dax
// Lọc bảng với điều kiện
High Value Sales = 
CALCULATE(
    SUM(Sales[Revenue]), 
    FILTER(Sales, Sales[Revenue] > 1000)
)

// Lọc với nhiều điều kiện
Q1_High_Sales = 
CALCULATE(
    SUM(Sales[Revenue]),
    FILTER(Sales, 
        Sales[Revenue] > 1000 && 
        Sales[Date] >= DATE(2024,1,1) && 
        Sales[Date] <= DATE(2024,3,31)
    )
)
```

**ALL / ALLEXCEPT / ALLSELECTED**
```dax
// ALL: Bỏ qua tất cả filter
Total Sales All = CALCULATE(SUM(Sales[Revenue]), ALL(Sales))

// ALLEXCEPT: Bỏ qua filter trừ các cột chỉ định
Sales by Category = CALCULATE(SUM(Sales[Revenue]), ALLEXCEPT(Sales, Sales[Category]))

// ALLSELECTED: Bỏ qua filter trong visual nhưng giữ filter bên ngoài
Percent of Selected = 
DIVIDE(
    SUM(Sales[Revenue]),
    CALCULATE(SUM(Sales[Revenue]), ALLSELECTED(Sales))
)
```

**CALCULATE / CALCULATETABLE**
```dax
// CALCULATE: Thay đổi filter context cho measure
Sales 2024 = CALCULATE(SUM(Sales[Revenue]), Sales[Year] = 2024)

// CALCULATETABLE: Thay đổi filter context cho table
High_Value_Table = CALCULATETABLE(Sales, Sales[Revenue] > 1000)
```

#### Hàm Lọc Nâng Cao

**KEEPFILTERS**
```dax
// Giữ filter hiện tại và thêm filter mới
Conservative_Filter = 
CALCULATE(
    SUM(Sales[Revenue]),
    KEEPFILTERS(Sales[Category] = "Electronics")
)
```

**REMOVEFILTERS**
```dax
// Xóa filter khỏi cột/bảng cụ thể
Total_Ignoring_Date = 
CALCULATE(
    SUM(Sales[Revenue]),
    REMOVEFILTERS(Sales[Date])
)
```

**VALUES / DISTINCT**
```dax
// VALUES: Trả về giá trị duy nhất trong filter context
Selected_Categories = VALUES(Products[Category])

// DISTINCT: Tương tự VALUES nhưng không bao gồm blank
Distinct_Regions = DISTINCT(Sales[Region])
```

**HASONEVALUE / ISFILTERED**
```dax
// HASONEVALUE: Kiểm tra có đúng 1 giá trị được chọn
Single_Product_Name = 
IF(
    HASONEVALUE(Products[ProductName]),
    VALUES(Products[ProductName]),
    "Multiple Products"
)

// ISFILTERED: Kiểm tra cột có bị filter không
Filter_Status = 
IF(
    ISFILTERED(Products[Category]),
    "Category is filtered",
    "No category filter"
)
```

### 3. Hàm Thời Gian (Time Intelligence)

#### Hàm So Sánh Thời Gian

**Cùng Kỳ Năm Trước**
```dax
// SAMEPERIODLASTYEAR
Sales_LY = CALCULATE(SUM(Sales[Revenue]), SAMEPERIODLASTYEAR(Dates[Date]))

// PARALLELPERIOD  
Sales_LY_Parallel = CALCULATE(SUM(Sales[Revenue]), PARALLELPERIOD(Dates[Date], -12, MONTH))

// DATEADD
Sales_Previous_Month = CALCULATE(SUM(Sales[Revenue]), DATEADD(Dates[Date], -1, MONTH))
```

**Tính Toán Tích Lũy**
```dax
// TOTALYTD: Tổng từ đầu năm
YTD_Sales = TOTALYTD(SUM(Sales[Revenue]), Dates[Date])

// TOTALQTD: Tổng từ đầu quý  
QTD_Sales = TOTALQTD(SUM(Sales[Revenue]), Dates[Date])

// TOTALMTD: Tổng từ đầu tháng
MTD_Sales = TOTALMTD(SUM(Sales[Revenue]), Dates[Date])

// DATESYTD: Tạo filter từ đầu năm
YTD_Custom = CALCULATE(SUM(Sales[Revenue]), DATESYTD(Dates[Date]))
```

**Hàm Khoảng Thời Gian**
```dax
// DATESBETWEEN: Khoảng thời gian tùy chỉnh
Sales_Last_30_Days = 
CALCULATE(
    SUM(Sales[Revenue]),
    DATESBETWEEN(
        Dates[Date],
        TODAY() - 30,
        TODAY()
    )
)

// DATESINPERIOD: N ngày/tháng/năm từ ngày bắt đầu
Sales_Last_6_Months = 
CALCULATE(
    SUM(Sales[Revenue]),
    DATESINPERIOD(Dates[Date], MAX(Dates[Date]), -6, MONTH)
)
```

#### Hàm Thời Gian Nâng Cao

**FIRSTDATE / LASTDATE**
```dax
// Ngày đầu/cuối trong filter context
First_Sale_Date = FIRSTDATE(Sales[Date])
Last_Sale_Date = LASTDATE(Sales[Date])

// FIRSTNONBLANK / LASTNONBLANK
First_Sale_Amount = FIRSTNONBLANK(Sales[Date], SUM(Sales[Revenue]))
```

**PREVIOUSMONTH / NEXTMONTH**
```dax
// Tháng trước/sau
Previous_Month_Sales = CALCULATE(SUM(Sales[Revenue]), PREVIOUSMONTH(Dates[Date]))
Next_Month_Target = CALCULATE(SUM(Targets[Amount]), NEXTMONTH(Dates[Date]))
```

**OPENINGBALANCE / CLOSINGBALANCE**
```dax
// Số dư đầu/cuối kỳ (cho dữ liệu semi-additive)
Opening_Inventory = OPENINGBALANCEMONTH(SUM(Inventory[Quantity]), Dates[Date])
Closing_Balance = CLOSINGBALANCEMONTH(SUM(Account[Balance]), Dates[Date])
```

### 4. Hàm Văn Bản (Text Functions)

#### Xử Lý Chuỗi Cơ Bản

**Nối Chuỗi**
```dax
// CONCATENATE: Nối 2 chuỗi
Full_Name = CONCATENATE(Customer[FirstName], " " & Customer[LastName])

// Toán tử &: Nối nhiều chuỗi
Address = Customer[Street] & ", " & Customer[City] & ", " & Customer[State]

// CONCATENATEX: Nối chuỗi từ bảng với delimiter
Product_List = CONCATENATEX(Products, Products[ProductName], ", ")
```

**Trích Xuất Chuỗi**
```dax
// LEFT/RIGHT/MID: Trích xuất từ vị trí
Product_Code = LEFT(Products[SKU], 3)
Product_Type = RIGHT(Products[SKU], 2) 
Product_Category = MID(Products[SKU], 4, 3)

// LEN: Độ dài chuỗi
Name_Length = LEN(Customer[CustomerName])
```

**Tìm Kiếm và Thay Thế**
```dax
// FIND/SEARCH: Tìm vị trí substring
At_Position = FIND("@", Customer[Email])
Domain_Position = SEARCH(".", Customer[Email], FIND("@", Customer[Email]))

// SUBSTITUTE/REPLACE: Thay thế chuỗi
Clean_Phone = SUBSTITUTE(SUBSTITUTE(Customer[Phone], "(", ""), ")", "")
Updated_Code = REPLACE(Product[Code], 1, 2, "XX")
```

#### Xử Lý Văn Bản Nâng Cao

**Chuyển Đổi Case**
```dax
// UPPER/LOWER/PROPER: Chuyển đổi case
Customer_Upper = UPPER(Customer[Name])
Product_Lower = LOWER(Product[Description])
Title_Case = PROPER(Customer[Name])
```

**Làm Sạch Dữ Liệu**
```dax
// TRIM: Xóa khoảng trắng thừa
Clean_Name = TRIM(Customer[Name])

// Xóa ký tự đặc biệt
Clean_Text = SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(
    Customer[Name], "!", ""), "@", ""), "#", "")
```

**Format Chuỗi**
```dax
// FORMAT: Định dạng số thành chuỗi
Formatted_Revenue = FORMAT(SUM(Sales[Revenue]), "$#,##0.00")
Formatted_Date = FORMAT(TODAY(), "DD/MM/YYYY")
Formatted_Percent = FORMAT(DIVIDE([Profit], [Revenue]), "0.00%")
```

### 5. Hàm Logic (Logical Functions)

#### Hàm Điều Kiện Cơ Bản

**IF / IF.EAGER**
```dax
// IF cơ bản
Sales_Category = IF(Sales[Revenue] > 1000, "High", "Low")

// IF lồng nhau
Performance_Rating = 
IF(Sales[Revenue] > 5000, "Excellent",
    IF(Sales[Revenue] > 2000, "Good",
        IF(Sales[Revenue] > 500, "Average", "Poor")
    )
)

// IF.EAGER: Đánh giá tất cả branches (hiếm khi dùng)
Eager_Result = IF.EAGER(Sales[Revenue] > 1000, "High", "Low")
```

**SWITCH**
```dax
// SWITCH với exact matches
Region_Priority = 
SWITCH(
    Sales[Region],
    "North", "High",
    "South", "Medium", 
    "East", "Medium",
    "West", "Low",
    "Unknown"
)

// SWITCH với TRUE() cho multiple conditions
Customer_Segment = 
SWITCH(
    TRUE(),
    [Customer_Revenue] > 10000, "VIP",
    [Customer_Revenue] > 5000, "Premium", 
    [Customer_Revenue] > 1000, "Standard",
    "Basic"
)
```

#### Hàm Logic Nâng Cao

**AND / OR / NOT**
```dax
// AND: Tất cả điều kiện đúng
High_Value_Local = 
IF(
    AND(Sales[Revenue] > 1000, Sales[Region] = "Local"),
    "Yes", "No"
)

// OR: Ít nhất một điều kiện đúng  
Priority_Customer = 
IF(
    OR(Sales[Revenue] > 5000, Customer[VIP_Status] = "Yes"),
    "Priority", "Standard"
)

// NOT: Phủ định điều kiện
Non_VIP = IF(NOT(Customer[VIP_Status] = "Yes"), "Regular", "VIP")
```

**IFERROR / ISERROR / ISBLANK**
```dax
// IFERROR: Xử lý lỗi
Safe_Division = IFERROR(DIVIDE(Sales[Profit], Sales[Revenue]), 0)

// ISERROR: Kiểm tra lỗi
Has_Error = ISERROR(DIVIDE(Sales[Profit], Sales[Revenue]))

// ISBLANK: Kiểm tra giá trị blank
Is_Missing = ISBLANK(Customer[Phone])
```

### 6. Hàm Quan Hệ (Relationship Functions)

#### Hàm Related

**RELATED / RELATEDTABLE**
```dax
// RELATED: Lấy giá trị từ bảng liên quan (many-to-one)
Product_Category = RELATED(Products[Category])
Customer_Segment = RELATED(Customers[Segment])

// RELATEDTABLE: Lấy bảng liên quan (one-to-many)
Customer_Orders = RELATEDTABLE(Sales)
Product_Sales = RELATEDTABLE(Sales)
```

#### Hàm Lookup

**LOOKUPVALUE**
```dax
// LOOKUPVALUE: Tìm kiếm giá trị
Product_Price = 
LOOKUPVALUE(
    Products[Price],
    Products[ProductID], Sales[ProductID]
)

// LOOKUPVALUE với multiple conditions
Customer_Discount = 
LOOKUPVALUE(
    Discounts[Rate],
    Discounts[CustomerID], Sales[CustomerID],
    Discounts[ProductCategory], RELATED(Products[Category])
)
```

### 7. Hàm Bảng (Table Functions)

#### Tạo và Thao Tác Bảng

**DATATABLE**
```dax
// Tạo bảng static
Status_Table = 
DATATABLE(
    "Status", STRING,
    "Priority", INTEGER,
    {
        {"High", 1},
        {"Medium", 2}, 
        {"Low", 3}
    }
)
```

**UNION / INTERSECT / EXCEPT**
```dax
// UNION: Hợp 2 bảng
All_Dates = UNION(Sales[Date], Returns[Date])

// INTERSECT: Giao 2 bảng  
Common_Customers = INTERSECT(VALUES(Sales[CustomerID]), VALUES(Returns[CustomerID]))

// EXCEPT: Hiệu 2 bảng
Customers_No_Returns = EXCEPT(VALUES(Sales[CustomerID]), VALUES(Returns[CustomerID]))
```

**CROSSJOIN / NATURALINNERJOIN**
```dax
// CROSSJOIN: Tích Cartesian
All_Combinations = CROSSJOIN(VALUES(Products[Category]), VALUES(Sales[Region]))

// NATURALINNERJOIN: Inner join tự động
Customer_Sales = NATURALINNERJOIN(Customers, Sales)
```

#### Hàm Sắp Xếp và Chọn Lọc

**TOPN**
```dax
// Top N records
Top_5_Products = 
TOPN(
    5,
    Products,
    [Total Sales],
    DESC
)

// Top N với ties
Top_Customers_With_Ties = 
TOPN(
    10,
    ADDCOLUMNS(
        VALUES(Sales[CustomerID]),
        "Customer Sales", [Total Sales]
    ),
    [Customer Sales],
    DESC
)
```

**SAMPLE**
```dax
// Lấy mẫu ngẫu nhiên
Random_Customers = SAMPLE(10, Customers, Customers[CustomerID])
```

### 8. Hàm Toán Học và Thống Kê

#### Hàm Toán Học Cơ Bản

**Phép Tính Cơ Bản**
```dax
// DIVIDE: Chia an toàn
Profit_Margin = DIVIDE(Sales[Profit], Sales[Revenue], 0)

// ABS: Giá trị tuyệt đối
Absolute_Variance = ABS([Actual] - [Budget])

// ROUND/ROUNDUP/ROUNDDOWN
Rounded_Revenue = ROUND(Sales[Revenue], -2) // Làm tròn đến hàng trăm
Rounded_Up = ROUNDUP(Sales[Revenue], 0)
Rounded_Down = ROUNDDOWN(Sales[Revenue], 0)
```

**Hàm Lũy Thừa và Logarit**
```dax
// POWER: Lũy thừa
Compound_Interest = [Principal] * POWER(1 + [Rate], [Years])

// SQRT: Căn bậc hai
Standard_Deviation = SQRT([Variance])

// LN/LOG/LOG10: Logarit
Natural_Log = LN([Value])
Log_Base_2 = LOG([Value], 2)
Log_Base_10 = LOG10([Value])
```

#### Hàm Thống Kê

**STDEV.S / STDEV.P**
```dax
// Độ lệch chuẩn mẫu/tổng thể
Revenue_StdDev_Sample = STDEV.S(Sales[Revenue])
Revenue_StdDev_Population = STDEV.P(Sales[Revenue])
```

**VAR.S / VAR.P**
```dax
// Phương sai mẫu/tổng thể  
Revenue_Variance_Sample = VAR.S(Sales[Revenue])
Revenue_Variance_Population = VAR.P(Sales[Revenue])
```

**GEOMEAN / HARMEAN**
```dax
// Trung bình nhân/điều hòa
Geometric_Mean = GEOMEAN(Sales[GrowthRate])
Harmonic_Mean = HARMEAN(Sales[Efficiency])
```

### 9. Hàm Ngày Tháng (Date/Time Functions)

#### Tạo và Trích Xuất Ngày

**DATE / TIME / DATETIME**
```dax
// Tạo ngày
Custom_Date = DATE(2024, 12, 31)
Custom_Time = TIME(14, 30, 0)
Custom_DateTime = DATETIME(2024, 12, 31, 14, 30, 0)

// Trích xuất thành phần
Sale_Year = YEAR(Sales[Date])
Sale_Month = MONTH(Sales[Date])
Sale_Day = DAY(Sales[Date])
Sale_Weekday = WEEKDAY(Sales[Date])
Sale_Quarter = "Q" & QUARTER(Sales[Date])
```

**Ngày Hiện Tại**
```dax
// NOW/TODAY
Current_DateTime = NOW()
Current_Date = TODAY()

// UTCNOW/UTCTODAY  
UTC_DateTime = UTCNOW()
UTC_Date = UTCTODAY()
```

#### Tính Toán Ngày

**DATEDIFF**
```dax
// Khoảng cách giữa 2 ngày
Days_Since_Order = DATEDIFF(Sales[OrderDate], TODAY(), DAY)
Months_Since_Launch = DATEDIFF(Products[LaunchDate], TODAY(), MONTH)
```

**EOMONTH / EDATE**
```dax
// Cuối tháng
End_Of_Month = EOMONTH(Sales[Date], 0)
End_Of_Next_Month = EOMONTH(Sales[Date], 1)

// Thêm tháng vào ngày
Date_Plus_3_Months = EDATE(Sales[Date], 3)
```

**WEEKNUM / YEARFRAC**
```dax
// Số tuần trong năm
Week_Number = WEEKNUM(Sales[Date])

// Phần của năm (cho tính lãi)
Year_Fraction = YEARFRAC(Sales[StartDate], Sales[EndDate])
```

---

## Hàm Nâng Cao và Chuyên Sâu

### 1. Iterator Functions và X Functions

#### Hiểu về Iterator Functions
Iterator functions duyệt qua từng hàng và thực hiện phép tính, sau đó tổng hợp kết quả.

```dax
// SUMX: Tính tổng expression cho mỗi hàng
Total_Profit = SUMX(Sales, Sales[Revenue] - Sales[Cost])

// AVERAGEX: Trung bình expression
Avg_Profit_Margin = AVERAGEX(Sales, DIVIDE(Sales[Profit], Sales[Revenue]))

// MINX/MAXX: Min/Max của expression
Max_Profit_Per_Order = MAXX(Sales, Sales[Revenue] - Sales[Cost])

// COUNTX: Đếm với điều kiện
High_Margin_Orders = COUNTX(Sales, IF(DIVIDE(Sales[Profit], Sales[Revenue]) > 0.3, 1))
```

#### Iterator Functions Nâng Cao

**PRODUCTX**
```dax
// Tích các giá trị
Compound_Growth = PRODUCTX(GrowthRates, 1 + GrowthRates[Rate]) - 1
```

**CONCATENATEX**
```dax
// Nối chuỗi với delimiter tùy chỉnh
Product_List = CONCATENATEX(
    FILTER(Products, Products[Category] = "Electronics"),
    Products[ProductName],
    ", ",
    Products[ProductName], ASC
)
```

### 2. Variables (VAR) và Return

Variables giúp code dễ đọc, hiệu suất tốt hơn và tránh tính toán lặp lại.

```dax
// Sử dụng VAR cơ bản
Customer_Metrics = 
VAR CustomerRevenue = SUM(Sales[Revenue])
VAR CustomerOrders = COUNT(Sales[OrderID])
VAR AvgOrderValue = DIVIDE(CustomerRevenue, CustomerOrders)
RETURN
    IF(
        AvgOrderValue > 500,
        "High Value",
        IF(AvgOrderValue > 200, "Medium Value", "Low Value")
    )

// VAR với table expressions
Top_Products_Revenue = 
VAR TopProducts = 
    TOPN(
        5,
        VALUES(Sales[ProductID]),
        [Total Sales],
        DESC
    )
VAR TopProductsRevenue = 
    SUMX(
        TopProducts,
        [Total Sales]
    )
RETURN TopProductsRevenue
```

### 3. EARLIER và Row Context

EARLIER được sử dụng để truy cập row context bên ngoài trong nested evaluations.

```dax
// Ranking sử dụng EARLIER
Product_Rank_In_Category = 
RANKX(
    FILTER(
        Products,
        Products[Category] = EARLIER(Products[Category])
    ),
    Products[Price],
    ,
    DESC
)

// Running total sử dụng EARLIER
Running_Total = 
SUMX(
    FILTER(
        Sales,
        Sales[Date] <= EARLIER(Sales[Date])
    ),
    Sales[Revenue]
)

// Percent of category
Percent_of_Category = 
DIVIDE(
    Sales[Revenue],
    SUMX(
        FILTER(
            Sales,
            Sales[Category] = EARLIER(Sales[Category])
        ),
        Sales[Revenue]
    )
)
```

### 4. Path Functions

Dành cho hierarchies và parent-child relationships.

```dax
// PATH: Tạo path từ child đến root
Employee_Path = PATH(Employees[EmployeeID], Employees[ManagerID])

// PATHLENGTH: Độ dài của path
Hierarchy_Level = PATHLENGTH([Employee_Path])

// PATHITEM: Lấy item tại vị trí cụ thể
Direct_Manager = LOOKUPVALUE(
    Employees[Name],
    Employees[EmployeeID],
    PATHITEM([Employee_Path], PATHLENGTH([Employee_Path]) - 1)
)

// PATHCONTAINS: Kiểm tra path có chứa value
Is_Under_Manager = PATHCONTAINS([Employee_Path], [SelectedManagerID])
```

### 5. Information Functions

**HASONEFILTER / HASONEVALUE**
```dax
// Dynamic title dựa trên filter context
Chart_Title = 
SWITCH(
    TRUE(),
    HASONEVALUE(Products[Category]), 
        "Sales for " & VALUES(Products[Category]),
    HASONEVALUE(Sales[Region]),
        "Sales for " & VALUES(Sales[Region]) & " Region", 
    "Total Sales - All Categories and Regions"
)
```

**USERRELATIONSHIP**
```dax
// Sử dụng relationship không active
Sales_by_Ship_Date = 
CALCULATE(
    SUM(Sales[Revenue]),
    USERELATIONSHIP(Sales[ShipDate], Dates[Date])
)

// So sánh Order Date vs Ship Date
Shipping_Delay_Analysis = 
VAR SalesByOrderDate = [Total Sales]
VAR SalesByShipDate = 
    CALCULATE(
        SUM(Sales[Revenue]),
        USERELATIONSHIP(Sales[ShipDate], Dates[Date])
    )
RETURN SalesByOrderDate - SalesByShipDate
```

**SELECTEDVALUE**
```dax
// Lấy giá trị được chọn, trả về alternative nếu nhiều giá trị
Selected_Product = SELECTEDVALUE(Products[ProductName], "Multiple Products")
Selected_Year = SELECTEDVALUE(Dates[Year], "All Years")

// Dùng trong conditional formatting
Traffic_Light = 
VAR SelectedThreshold = SELECTEDVALUE(Thresholds[Value], 100)
RETURN
    IF([Current Value] > SelectedThreshold, "Green", "Red")
```

---

## Context và Filter Context

### 1. Filter Context Transitions

#### Row Context to Filter Context
```dax
// Calculated Column - có row context tự nhiên
Customer_Total_Sales = 
CALCULATE(SUM(Sales[Revenue])) // CALCULATE chuyển row context thành filter context

// Measure - không có row context
Customer_Sales_Measure = 
SUMX(
    Customers,
    CALCULATE(SUM(Sales[Revenue])) // Iterator tạo row context, CALCULATE chuyển đổi
)
```

#### Filter Propagation
```dax
// Filter lan truyền qua relationships
// Filter trên Customers sẽ filter Sales thông qua relationship
Filtered_Customer_Sales = 
CALCULATE(
    SUM(Sales[Revenue]),
    Customers[Segment] = "Premium"
)

// Ngăn filter propagation
Sales_All_Customers = 
CALCULATE(
    SUM(Sales[Revenue]),
    ALL(Customers) // Bỏ filter trên Customers nhưng giữ filter khác
)
```

### 2. Context Transition Examples

```dax
// Calculated Column: Row context tự động
Revenue_Rank_Column = 
RANKX(ALL(Sales), Sales[Revenue],, DESC) // Sai - không có context transition

Revenue_Rank_Column_Correct = 
RANKX(ALL(Sales), CALCULATE(SUM(Sales[Revenue])),, DESC) // Đúng

// Measure: Cần iterator để tạo row context
Revenue_by_Customer = 
SUMX(
    VALUES(Sales[CustomerID]),
    CALCULATE(SUM(Sales[Revenue])) // Context transition xảy ra
)
```

### 3. Advanced Filter Context

**Filter Context Inheritance**
```dax
// Filter được kế thừa qua các levels
Quarter_Sales = 
VAR CurrentQuarter = SELECTEDVALUE(Dates[Quarter])
VAR QuarterSales = 
    CALCULATE(
        SUM(Sales[Revenue]),
        FILTER(ALL(Dates), Dates[Quarter] = CurrentQuarter)
    )
RETURN QuarterSales

// Expanded table context
Customer_Category_Sales = 
SUMX(
    SUMMARIZE(
        Sales,
        Sales[CustomerID],
        Products[Category]
    ),
    CALCULATE(SUM(Sales[Revenue]))
)
```

**Multiple Filter Context**
```dax
// Filters được kết hợp
Complex_Filter_Sales = 
CALCULATE(
    SUM(Sales[Revenue]),
    Sales[Region] = "North",           // Filter 1
    Products[Category] = "Electronics", // Filter 2 (qua relationship)
    Dates[Year] = 2024                 // Filter 3 (qua relationship)
)

// Override vs Add filters
Override_Filter = 
CALCULATE(
    SUM(Sales[Revenue]),
    ALL(Sales[Region]),      // Xóa filter hiện tại
    Sales[Region] = "South"  // Thêm filter mới
)

Keep_Filter = 
CALCULATE(
    SUM(Sales[Revenue]),
    KEEPFILTERS(Sales[Region] = "South") // Giữ filter hiện tại, intersect với filter mới
)
```

---

## Patterns và Best Practices

### 1. Common DAX Patterns

#### Time Intelligence Pattern
```dax
// Base measure
Sales = SUM(Sales[Revenue])

// YTD
Sales YTD = TOTALYTD([Sales], Dates[Date])

// Previous Year
Sales PY = CALCULATE([Sales], SAMEPERIODLASTYEAR(Dates[Date]))

// YTD Previous Year  
Sales YTD PY = CALCULATE([Sales YTD], SAMEPERIODLASTYEAR(Dates[Date]))

// Growth
Sales YoY Growth = [Sales] - [Sales PY]
Sales YoY Growth % = DIVIDE([Sales YoY Growth], [Sales PY])

// YTD Growth
Sales YTD Growth % = DIVIDE([Sales YTD] - [Sales YTD PY], [Sales YTD PY])
```

#### ABC Analysis Pattern
```dax
// Customer Revenue Rank
Customer_Revenue_Rank = 
RANKX(
    ALL(Customers[CustomerID]),
    [Customer Revenue],
    ,
    DESC,
    DENSE
)

// Total Customers
Total_Customers = DISTINCTCOUNT(Sales[CustomerID])

// Percentile
Customer_Percentile = DIVIDE([Customer_Revenue_Rank], [Total_Customers])

// ABC Classification
Customer_ABC = 
SWITCH(
    TRUE(),
    [Customer_Percentile] <= 0.8, "A",
    [Customer_Percentile] <= 0.95, "B",
    "C"
)
```

#### Cohort Analysis Pattern
```dax
// First Purchase Date per Customer
Customer_First_Purchase = 
CALCULATE(
    MIN(Sales[Date]),
    ALLEXCEPT(Sales, Sales[CustomerID])
)

// Cohort (First Purchase Month)
Customer_Cohort = 
FORMAT([Customer_First_Purchase], "YYYY-MM")

// Months Since First Purchase
Months_Since_First_Purchase = 
DATEDIFF([Customer_First_Purchase], MAX(Sales[Date]), MONTH)

// Cohort Size
Cohort_Size = 
CALCULATE(
    DISTINCTCOUNT(Sales[CustomerID]),
    Sales[Date] = [Customer_First_Purchase]
)

// Retention Rate
Retention_Rate = 
DIVIDE(
    DISTINCTCOUNT(Sales[CustomerID]),
    [Cohort_Size]
)
```

### 2. Performance Best Practices

#### Efficient Measures
```dax
// ❌ Inefficient - Multiple scans
Bad_Measure = 
SUM(Sales[Revenue]) + 
CALCULATE(SUM(Sales[Revenue]), Sales[Category] = "A") +
CALCULATE(SUM(Sales[Revenue]), Sales[Category] = "B")

// ✅ Efficient - Single scan with variables
Good_Measure = 
VAR TotalSales = SUM(Sales[Revenue])
VAR CategoryA = CALCULATE(SUM(Sales[Revenue]), Sales[Category] = "A")
VAR CategoryB = CALCULATE(SUM(Sales[Revenue]), Sales[Category] = "B") 
RETURN TotalSales + CategoryA + CategoryB

// ✅ Even better - Use SUMX with conditional logic
Best_Measure = 
SUMX(
    Sales,
    Sales[Revenue] * 
    (1 + IF(Sales[Category] = "A", 1, 0) + IF(Sales[Category] = "B", 1, 0))
)
```

#### Memory Optimization
```dax
// ❌ Memory intensive - Creates large intermediate table
Heavy_Calculation = 
SUMX(
    CROSSJOIN(Products, Customers),
    [Some_Complex_Calculation]
)

// ✅ Memory efficient - Filter first
Light_Calculation = 
SUMX(
    FILTER(
        CROSSJOIN(Products, Customers),
        [Some_Condition] = TRUE()
    ),
    [Some_Complex_Calculation]
)
```

### 3. Error Handling Patterns

#### Comprehensive Error Handling
```dax
Safe_Division_Advanced = 
VAR Numerator = SUM(Sales[Profit])
VAR Denominator = SUM(Sales[Revenue])
VAR Result = 
    SWITCH(
        TRUE(),
        ISBLANK(Numerator) || ISBLANK(Denominator), BLANK(),
        Denominator = 0, BLANK(),
        ISERROR(DIVIDE(Numerator, Denominator)), BLANK(),
        DIVIDE(Numerator, Denominator)
    )
RETURN Result
```

#### Null and Blank Handling
```dax
Customer_Segment_Safe = 
VAR CustomerRevenue = [Customer Revenue]
VAR Segment = 
    IF(
        ISBLANK(CustomerRevenue) || CustomerRevenue = 0,
        "No Sales",
        SWITCH(
            TRUE(),
            CustomerRevenue > 10000, "VIP",
            CustomerRevenue > 5000, "Premium",
            CustomerRevenue > 1000, "Standard",
            "Basic"
        )
    )
RETURN Segment
```

### 4. Dynamic Calculations

#### Parameter-Driven Calculations
```dax
// Sử dụng với Parameter table
Dynamic_Time_Calc = 
VAR SelectedPeriod = SELECTEDVALUE(Parameters[Period], "MTD")
VAR Result = 
    SWITCH(
        SelectedPeriod,
        "MTD", TOTALMTD([Sales], Dates[Date]),
        "QTD", TOTALQTD([Sales], Dates[Date]), 
        "YTD", TOTALYTD([Sales], Dates[Date]),
        [Sales]
    )
RETURN Result
```

#### Field Parameter Pattern
```dax
// Dynamic measure selection
Dynamic_Measure = 
VAR SelectedMeasure = SELECTEDVALUE(MeasureSelection[Measure])
VAR Result = 
    SWITCH(
        SelectedMeasure,
        "Revenue", [Total Revenue],
        "Profit", [Total Profit],
        "Orders", [Total Orders],
        "Customers", [Total Customers],
        BLANK()
    )
RETURN Result
```

---

## Tình Huống Thực Tế

### 1. Phân Tích Bán Hàng (Sales Analytics)

#### Dashboard Bán Hàng Tổng Quan
```dax
// KPIs chính
Total_Revenue = SUM(Sales[Revenue])
Total_Orders = DISTINCTCOUNT(Sales[OrderID])
Total_Customers = DISTINCTCOUNT(Sales[CustomerID])
Avg_Order_Value = DIVIDE([Total_Revenue], [Total_Orders])

// So sánh với kỳ trước
Revenue_LY = CALCULATE([Total_Revenue], SAMEPERIODLASTYEAR(Dates[Date]))
Revenue_Growth = [Total_Revenue] - [Revenue_LY]
Revenue_Growth_% = DIVIDE([Revenue_Growth], [Revenue_LY])

// Trending
Revenue_Trend = 
VAR Current = [Total_Revenue]
VAR Previous = CALCULATE([Total_Revenue], DATEADD(Dates[Date], -1, MONTH))
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Previous), "No Data",
        Current > Previous * 1.05, "📈 Strong Growth",
        Current > Previous, "↗️ Growth", 
        Current > Previous * 0.95, "➡️ Stable",
        "📉 Decline"
    )
```

#### Phân Tích Sản Phẩm
```dax
// Product Performance
Product_Revenue_Rank = 
RANKX(
    ALLSELECTED(Products[ProductName]),
    [Total_Revenue],
    ,
    DESC
)

// Pareto Analysis (80/20 rule)
Product_Revenue_Cumulative = 
VAR CurrentProduct = SELECTEDVALUE(Products[ProductName])
VAR ProductsToInclude = 
    FILTER(
        ALLSELECTED(Products[ProductName]),
        [Product_Revenue_Rank] <= [Product_Revenue_Rank]
    )
RETURN
    SUMX(ProductsToInclude, [Total_Revenue])

Product_Revenue_Cumulative_% = 
DIVIDE(
    [Product_Revenue_Cumulative],
    CALCULATE([Total_Revenue], ALLSELECTED(Products))
)

// ABC Classification
Product_ABC = 
SWITCH(
    TRUE(),
    [Product_Revenue_Cumulative_%] <= 0.8, "A",
    [Product_Revenue_Cumulative_%] <= 0.95, "B", 
    "C"
)
```

### 2. Phân Tích Khách Hàng (Customer Analytics)

#### RFM Analysis
```dax
// Recency (Days since last purchase)
Customer_Recency = 
DATEDIFF(
    CALCULATE(MAX(Sales[Date]), ALLEXCEPT(Sales, Sales[CustomerID])),
    TODAY(),
    DAY
)

// Frequency (Number of orders)
Customer_Frequency = 
CALCULATE(
    DISTINCTCOUNT(Sales[OrderID]),
    ALLEXCEPT(Sales, Sales[CustomerID])
)

// Monetary (Total spent)
Customer_Monetary = 
CALCULATE(
    SUM(Sales[Revenue]),
    ALLEXCEPT(Sales, Sales[CustomerID])
)

// RFM Scores (1-5 scale)
R_Score = 
VAR RecencyPercentile = 
    DIVIDE(
        RANKX(
            ALL(Customers[CustomerID]),
            [Customer_Recency],
            ,
            ASC
        ),
        DISTINCTCOUNT(ALL(Customers[CustomerID]))
    )
RETURN
    SWITCH(
        TRUE(),
        RecencyPercentile <= 0.2, 5,
        RecencyPercentile <= 0.4, 4,
        RecencyPercentile <= 0.6, 3,
        RecencyPercentile <= 0.8, 2,
        1
    )

// Similar logic for F_Score and M_Score...

// RFM Segment
RFM_Segment = 
VAR RFMString = [R_Score] & [F_Score] & [M_Score]
RETURN
    SWITCH(
        TRUE(),
        RFMString IN {"555", "554", "544", "545", "454", "455", "445"}, "Champions",
        RFMString IN {"543", "444", "435", "355", "354", "345", "344", "335"}, "Loyal Customers",
        RFMString IN {"512", "511", "422", "421", "412", "411", "311"}, "Potential Loyalists",
        RFMString IN {"533", "532", "531", "523", "522", "521", "515", "514", "513", "425", "424", "413", "414", "415", "315", "314", "313"}, "New Customers",
        RFMString IN {"155", "154", "144", "214", "215", "115", "114"}, "Champions",
        RFMString IN {"155", "154", "144", "214", "215", "115", "114"}, "Can't Lose Them",
        "Others"
    )
```

#### Customer Lifetime Value
```dax
// Average Order Value per Customer
Customer_AOV = 
DIVIDE(
    CALCULATE(SUM(Sales[Revenue]), ALLEXCEPT(Sales, Sales[CustomerID])),
    CALCULATE(DISTINCTCOUNT(Sales[OrderID]), ALLEXCEPT(Sales, Sales[CustomerID]))
)

// Purchase Frequency (orders per month)
Customer_Purchase_Frequency = 
VAR FirstPurchase = 
    CALCULATE(MIN(Sales[Date]), ALLEXCEPT(Sales, Sales[CustomerID]))
VAR LastPurchase = 
    CALCULATE(MAX(Sales[Date]), ALLEXCEPT(Sales, Sales[CustomerID]))
VAR MonthsActive = 
    DATEDIFF(FirstPurchase, LastPurchase, MONTH) + 1
VAR TotalOrders = 
    CALCULATE(DISTINCTCOUNT(Sales[OrderID]), ALLEXCEPT(Sales, Sales[CustomerID]))
RETURN DIVIDE(TotalOrders, MonthsActive)

// Customer Lifespan (estimated months)
Customer_Lifespan = 
VAR PurchaseFreq = [Customer_Purchase_Frequency]
RETURN IF(PurchaseFreq > 0, DIVIDE(1, PurchaseFreq), BLANK())

// CLV Calculation
Customer_LTV = [Customer_AOV] * [Customer_Purchase_Frequency] * [Customer_Lifespan]
```

### 3. Phân Tích Tài Chính (Financial Analytics)

#### Profit & Loss Analysis
```dax
// Revenue Recognition
Recognized_Revenue = 
VAR ServiceStartDate = Sales[ServiceStartDate]
VAR ServiceEndDate = Sales[ServiceEndDate]
VAR CurrentMonthStart = EOMONTH(TODAY(), -1) + 1
VAR CurrentMonthEnd = EOMONTH(TODAY(), 0)
RETURN
    SUMX(
        Sales,
        IF(
            AND(
                Sales[ServiceStartDate] <= CurrentMonthEnd,
                Sales[ServiceEndDate] >= CurrentMonthStart
            ),
            VAR OverlapStart = MAX(ServiceStartDate, CurrentMonthStart)
            VAR OverlapEnd = MIN(ServiceEndDate, CurrentMonthEnd)
            VAR OverlapDays = OverlapEnd - OverlapStart + 1
            VAR TotalServiceDays = ServiceEndDate - ServiceStartDate + 1
            VAR RevenuePerDay = DIVIDE(Sales[Revenue], TotalServiceDays)
            RETURN RevenuePerDay * OverlapDays,
            0
        )
    )

// Cost Allocation
Allocated_Cost = 
SUMX(
    Sales,
    VAR DirectCost = Sales[DirectCost]
    VAR IndirectCostRate = 0.15 // 15% của revenue
    VAR IndirectCost = Sales[Revenue] * IndirectCostRate
    RETURN DirectCost + IndirectCost
)

// Margin Analysis
Gross_Margin = [Recognized_Revenue] - [Allocated_Cost]
Gross_Margin_% = DIVIDE([Gross_Margin], [Recognized_Revenue])
```

#### Budget vs Actual Analysis
```dax
// Variance Analysis
Budget_Variance = [Actual_Revenue] - [Budget_Revenue]
Budget_Variance_% = DIVIDE([Budget_Variance], [Budget_Revenue])

// Forecast Accuracy
Forecast_Accuracy = 
1 - ABS(DIVIDE([Budget_Variance], [Budget_Revenue]))

// Traffic Light KPI
Budget_Status = 
SWITCH(
    TRUE(),
    [Budget_Variance_%] >= 0.05, "🟢 Above Target",
    [Budget_Variance_%] >= -0.05, "🟡 On Target",
    "🔴 Below Target"
)

// YTD vs Budget
YTD_Actual = TOTALYTD([Actual_Revenue], Dates[Date])
YTD_Budget = TOTALYTD([Budget_Revenue], Dates[Date])
YTD_Variance_% = DIVIDE([YTD_Actual] - [YTD_Budget], [YTD_Budget])
```

### 4. Phân Tích Vận Hành (Operational Analytics)

#### Inventory Analysis
```dax
// Stock Levels
Current_Stock = 
LASTNONBLANKVALUE(
    Dates[Date],
    SUM(Inventory[Quantity])
)

// Stock Turnover
Inventory_Turnover = 
DIVIDE(
    CALCULATE(SUM(Sales[QuantitySold]), DATESYTD(Dates[Date])),
    CALCULATE(AVERAGE(Inventory[Quantity]), DATESYTD(Dates[Date]))
)

// Days Sales Outstanding
Days_In_Inventory = DIVIDE(365, [Inventory_Turnover])

// Reorder Point Calculation
Reorder_Point = 
VAR LeadTime = RELATED(Products[LeadTimeDays])
VAR DailyUsage = 
    DIVIDE(
        CALCULATE(SUM(Sales[QuantitySold]), DATESINPERIOD(Dates[Date], TODAY(), -30, DAY)),
        30
    )
VAR SafetyStock = DailyUsage * 5 // 5 days safety stock
RETURN (DailyUsage * LeadTime) + SafetyStock
```

#### Quality Metrics
```dax
// Defect Rate
Defect_Rate = 
DIVIDE(
    CALCULATE(COUNT(Production[BatchID]), Production[Status] = "Defective"),
    COUNT(Production[BatchID])
)

// First Pass Yield
First_Pass_Yield = 
DIVIDE(
    CALCULATE(COUNT(Production[BatchID]), Production[FirstPassStatus] = "Pass"),
    COUNT(Production[BatchID])
)

// Overall Equipment Effectiveness (OEE)
Availability = 
DIVIDE([Actual_Runtime], [Planned_Runtime])

Performance = 
DIVIDE([Actual_Output], [Theoretical_Max_Output])

Quality = 1 - [Defect_Rate]

OEE = [Availability] * [Performance] * [Quality]
```

### 5. HR Analytics

#### Employee Performance
```dax
// Employee Utilization
Employee_Utilization = 
DIVIDE(
    CALCULATE(SUM(Timesheet[BillableHours]), ALLEXCEPT(Timesheet, Timesheet[EmployeeID])),
    CALCULATE(SUM(Timesheet[TotalHours]), ALLEXCEPT(Timesheet, Timesheet[EmployeeID]))
)

// Revenue per Employee
Revenue_per_Employee = 
DIVIDE(
    CALCULATE(SUM(Sales[Revenue]), ALLEXCEPT(Sales, Sales[EmployeeID])),
    1
)

// Employee Ranking
Employee_Performance_Rank = 
RANKX(
    ALL(Employees[EmployeeID]),
    [Revenue_per_Employee],
    ,
    DESC
)
```

#### Attrition Analysis
```dax
// Turnover Rate
Turnover_Rate = 
VAR EmployeesStart = 
    CALCULATE(
        DISTINCTCOUNT(Employees[EmployeeID]),
        Employees[StartDate] <= EOMONTH(TODAY(), -1)
    )
VAR EmployeesLeft = 
    CALCULATE(
        DISTINCTCOUNT(Employees[EmployeeID]),
        Employees[EndDate] >= EOMONTH(TODAY(), -1) + 1,
        Employees[EndDate] <= EOMONTH(TODAY(), 0)
    )
RETURN DIVIDE(EmployeesLeft, EmployeesStart)

// Average Tenure
Average_Tenure = 
AVERAGEX(
    Employees,
    DATEDIFF(
        Employees[StartDate],
        COALESCE(Employees[EndDate], TODAY()),
        MONTH
    )
)
```

---

## Debugging và Tối Ưu Hóa

### 1. Common DAX Errors và Cách Khắc Phục

#### Circular Dependency
```dax
// ❌ Circular reference
Measure_A = [Measure_B] + 100
Measure_B = [Measure_A] - 50

// ✅ Solution: Break the circular reference
Base_Value = SUM(Sales[Revenue])
Measure_A = [Base_Value] + 100  
Measure_B = [Base_Value] - 50
```

#### Context Transition Issues
```dax
// ❌ Missing context transition
Wrong_Ranking = 
SUMX(
    Products,
    RANKX(ALL(Products), SUM(Sales[Revenue]),, DESC) // Sẽ trả về cùng 1 giá trị
)

// ✅ Correct context transition
Correct_Ranking = 
SUMX(
    Products,
    RANKX(ALL(Products), CALCULATE(SUM(Sales[Revenue])),, DESC)
)
```

#### Performance Issues
```dax
// ❌ Expensive calculation
Slow_Measure = 
SUMX(
    Sales,
    CALCULATE(
        SUM(Products[Cost]),
        Products[ProductID] = Sales[ProductID]
    )
)

// ✅ Use RELATED instead
Fast_Measure = SUMX(Sales, Sales[Quantity] * RELATED(Products[Cost]))
```

### 2. Performance Optimization Techniques

#### Reduce Cardinality
```dax
// ❌ High cardinality operation
Expensive_Calc = 
SUMX(
    CROSSJOIN(
        VALUES(Sales[CustomerID]),
        VALUES(Products[ProductID])
    ),
    [Some_Calculation]
)

// ✅ Filter to reduce combinations
Optimized_Calc = 
SUMX(
    FILTER(
        CROSSJOIN(
            VALUES(Sales[CustomerID]),
            VALUES(Products[ProductID])
        ),
        NOT(ISBLANK([Some_Calculation]))
    ),
    [Some_Calculation]
)
```

#### Materialize Common Calculations
```dax
// ❌ Repeated complex calculations
Complex_Measure = 
VAR ComplexCalc1 = SUMX(Sales, Sales[Revenue] * Sales[Margin] * Sales[Discount])
VAR ComplexCalc2 = SUMX(Sales, Sales[Revenue] * Sales[Margin] * Sales[Discount])
RETURN ComplexCalc1 + ComplexCalc2

// ✅ Calculate once, reuse
Optimized_Complex_Measure = 
VAR ComplexCalc = SUMX(Sales, Sales[Revenue] * Sales[Margin] * Sales[Discount])
RETURN ComplexCalc + ComplexCalc // Reuse the variable
```

### 3. Debugging Tools và Techniques

#### Using Variables for Debugging
```dax
Debug_Measure = 
VAR Step1 = SUM(Sales[Revenue])
VAR Step2 = CALCULATE(SUM(Sales[Revenue]), Sales[Category] = "Electronics")
VAR Step3 = DIVIDE(Step2, Step1)
VAR DebugInfo = 
    "Total: " & Step1 & 
    " | Electronics: " & Step2 & 
    " | Ratio: " & Step3
RETURN 
    IF(
        SELECTEDVALUE(DebugTable[ShowDebug], FALSE()),
        DebugInfo,
        Step3
    )
```

#### Error Checking Pattern
```dax
Robust_Measure = 
VAR ErrorCheck = 
    SWITCH(
        TRUE(),
        ISBLANK(SUM(Sales[Revenue])), "No Revenue Data",
        COUNTROWS(Sales) = 0, "No Sales Data",
        NOT(HASONEVALUE(Products[ProductID])) && [Detail_Level] = "Product", "Multiple Products Selected",
        ""
    )
VAR Result = 
    IF(
        ErrorCheck = "",
        [Your_Actual_Calculation],
        ERROR(ErrorCheck)
    )
RETURN Result
```

### 4. Memory và Storage Optimization

#### Calculated Columns vs Measures
```dax
// ❌ Calculated Column (uses memory)
Sales[Profit Margin] = DIVIDE(Sales[Profit], Sales[Revenue])

// ✅ Measure (calculated on demand)
Profit Margin = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]))
```

#### Efficient Data Types
```dax
// ❌ Text for numeric codes
Customer[Customer_Code] = "CUST" & Customer[Customer_Number]

// ✅ Keep as integer, format in visuals
Customer_Code_Display = "CUST" & FORMAT(Customer[Customer_Number], "0000")
```

---

## Tài Nguyên Hỗ Trợ

### 1. Tài Liệu và Học Tập
- **Microsoft Learn**: https://learn.microsoft.com/en-us/power-bi/
- **DAX Guide**: https://dax.guide/ - Tham khảo đầy đủ các hàm DAX
- **SQLBI**: https://www.sqlbi.com/ - Chuyên gia hàng đầu về DAX
- **PowerBI.tips**: https://powerbi.tips/ - Tips và tricks thực tế

### 2. Tools Hỗ Trợ
- **DAX Studio**: Công cụ debug và optimize DAX queries
- **Tabular Editor**: Advanced model editing tool
- **ALM Toolkit**: So sánh và deploy models
- **Power BI Helper**: Chrome extension cho Power BI

### 3. Community và Diễn Đàn
- **Power BI Community**: https://community.powerbi.com/
- **Reddit r/PowerBI**: https://www.reddit.com/r/PowerBI/
- **Stack Overflow**: Tag [powerbi] và [dax]
- **LinkedIn Power BI Groups**: Networking và knowledge sharing

### 4. Best Practice Resources
- **Microsoft Power BI Adoption Framework**
- **Power BI Implementation Planning**
- **DAX Patterns**: https://www.daxpatterns.com/
- **Enterprise DNA**: https://enterprisedna.co/

### 5. Certification và Training
- **Microsoft Certified: Data Analyst Associate**
- **Microsoft Certified: Power BI Data Analyst Associate**
- **SQLBI Courses**: Advanced DAX và modeling
- **Enterprise DNA University**: Comprehensive Power BI training

---

## Kết Luận

DAX là ngôn ngữ mạnh mẽ và linh hoạt cho phép bạn thực hiện các phân tích phức tạp trong Power BI. Thành thạo DAX đòi hỏi:

1. **Hiểu rõ concepts cơ bản**: Context, relationships, evaluation order
2. **Thực hành thường xuyên**: Bắt đầu với các hàm đơn giản, tiến dần lên phức tạp
3. **Học từ patterns**: Sử dụng các mẫu đã được kiểm chứng
4. **Tối ưu hóa performance**: Luôn nghĩ về hiệu suất và memory usage
5. **Debug effectively**: Sử dụng variables và error handling
6. **Tham gia community**: Học hỏi và chia sẻ kinh nghiệm

### Next Steps
1. Thực hành với dataset thực tế của bạn
2. Tham gia các khóa học nâng cao
3. Đóng góp vào community
4. Xây dựng portfolio các solutions DAX
5. Chuẩn bị cho certification

### Final Tips
- **Start simple**: Bắt đầu với measures cơ bản trước khi chuyển sang phức tạp
- **Document your code**: Sử dụng comments và meaningful names  
- **Test thoroughly**: Kiểm tra kết quả với nhiều filter contexts khác nhau
- **Stay updated**: DAX được cập nhật thường xuyên với features mới
- **Think like SQL**: Nhiều concepts tương tự SQL nhưng có điểm khác biệt quan trọng

Chúc bạn thành công trong hành trình làm chủ DAX và Power BI!

---

## Phụ Lục

### A. Cheat Sheet - Hàm DAX Quan Trọng Nhất

#### Top 20 Hàm Phải Biết
```dax
// 1. SUM - Tổng cơ bản
Total_Sales = SUM(Sales[Revenue])

// 2. CALCULATE - Thay đổi filter context  
Sales_2024 = CALCULATE(SUM(Sales[Revenue]), Sales[Year] = 2024)

// 3. SUMX - Iterator tính tổng
Total_Profit = SUMX(Sales, Sales[Revenue] - Sales[Cost])

// 4. FILTER - Lọc dữ liệu
High_Value = CALCULATE(SUM(Sales[Revenue]), FILTER(Sales, Sales[Revenue] > 1000))

// 5. ALL - Bỏ qua filter
Total_All = CALCULATE(SUM(Sales[Revenue]), ALL(Sales))

// 6. VALUES - Giá trị duy nhất trong context
Selected_Regions = VALUES(Sales[Region])

// 7. RELATED - Lấy dữ liệu từ bảng liên quan
Product_Category = RELATED(Products[Category])

// 8. DIVIDE - Chia an toàn
Profit_Margin = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]))

// 9. IF - Điều kiện
Status = IF(Sales[Revenue] > 1000, "High", "Low")

// 10. SWITCH - Nhiều điều kiện
Rating = SWITCH(Sales[Region], "North", "A", "South", "B", "C")

// 11. RANKX - Xếp hạng
Product_Rank = RANKX(ALL(Products), [Total_Sales],, DESC)

// 12. TOTALYTD - Tổng từ đầu năm
YTD_Sales = TOTALYTD(SUM(Sales[Revenue]), Dates[Date])

// 13. SAMEPERIODLASTYEAR - Cùng kỳ năm trước
Sales_LY = CALCULATE(SUM(Sales[Revenue]), SAMEPERIODLASTYEAR(Dates[Date]))

// 14. DISTINCTCOUNT - Đếm giá trị duy nhất
Unique_Customers = DISTINCTCOUNT(Sales[CustomerID])

// 15. AVERAGE - Trung bình
Avg_Order = AVERAGE(Sales[OrderValue])

// 16. MIN/MAX - Giá trị nhỏ nhất/lớn nhất
Min_Price = MIN(Products[Price])
Max_Price = MAX(Products[Price])

// 17. CONCATENATEX - Nối chuỗi
Product_List = CONCATENATEX(Products, Products[Name], ", ")

// 18. ISBLANK - Kiểm tra giá trị trống
Has_Data = IF(ISBLANK(SUM(Sales[Revenue])), "No Data", "Has Data")

// 19. FORMAT - Định dạng
Formatted_Sales = FORMAT(SUM(Sales[Revenue]), "$#,##0")

// 20. VAR/RETURN - Biến và trả về
Complex_Calc = 
VAR Revenue = SUM(Sales[Revenue])
VAR Cost = SUM(Sales[Cost])
RETURN Revenue - Cost
```

### B. Error Messages và Cách Khắc Phục

#### Lỗi Thường Gặp

**1. "Circular dependency detected"**
```dax
// Nguyên nhân: Measures tham chiếu lẫn nhau
// Giải pháp: Tạo base measure
Base_Revenue = SUM(Sales[Revenue])
Adjusted_Revenue = [Base_Revenue] * 1.1
```

**2. "The column '[Column]' either doesn't exist or doesn't have a relationship"**
```dax
// Nguyên nhân: Không có relationship hoặc sai tên cột
// Giải pháp: Kiểm tra relationship và tên cột
Correct_Reference = RELATED(Products[Category]) // Đảm bảo có relationship
```

**3. "A single value for column cannot be determined"**
```dax
// Nguyên nhân: Multiple values trong scalar context
// Giải pháp: Sử dụng aggregation function
Wrong_Way = Products[Price] // Lỗi nếu multiple products
Right_Way = SELECTEDVALUE(Products[Price], 0) // Hoặc SUM, AVERAGE, etc.
```

**4. "The syntax for '[Measure]' is incorrect"**
```dax
// Nguyên nhân: Sai cú pháp
// Giải pháp: Kiểm tra dấu ngoặc, dấu phẩy
Correct_Syntax = CALCULATE(SUM(Sales[Revenue]), Sales[Year] = 2024)
```

### C. Performance Tuning Checklist

#### Model Optimization
- [ ] Sử dụng Measures thay vì Calculated Columns khi có thể
- [ ] Loại bỏ columns không cần thiết
- [ ] Sử dụng appropriate data types
- [ ] Tạo proper relationships
- [ ] Implement date table
- [ ] Sử dụng star schema

#### DAX Optimization  
- [ ] Sử dụng DIVIDE thay vì phép chia thường
- [ ] Tránh FILTER trên large tables
- [ ] Sử dụng Variables để tránh tính toán lặp
- [ ] Prefer SUMX over complex CALCULATE chains
- [ ] Use RELATED instead of LOOKUPVALUE
- [ ] Minimize context transitions

#### Visual Optimization
- [ ] Limit data points trong visuals
- [ ] Sử dụng appropriate chart types
- [ ] Implement drill-through thay vì detailed visuals
- [ ] Use bookmarks for navigation
- [ ] Optimize slicer interactions

### D. DAX Naming Conventions

#### Best Practices
```dax
// Measures: Descriptive names with spaces
Total Sales = SUM(Sales[Revenue])
Sales Growth % = DIVIDE([Sales] - [Sales LY], [Sales LY])

// Calculated Columns: Table prefix
Products[Full Product Name] = Products[Brand] & " - " & Products[Name]

// Variables: Clear, concise names
VAR CurrentMonthSales = CALCULATE(SUM(Sales[Revenue]), DATESMTD(Dates[Date]))
VAR PreviousMonthSales = CALCULATE(SUM(Sales[Revenue]), DATEADD(Dates[Date], -1, MONTH))

// Tables: Plural nouns
Sales, Products, Customers, Dates

// Columns: Singular nouns without spaces
Sales[OrderDate], Products[ProductName], Customers[CustomerID]
```

### E. Testing Framework cho DAX

#### Unit Testing Pattern
```dax
// Test Measure
Test_Revenue_Calculation = 
VAR ExpectedResult = 1000000
VAR ActualResult = [Total Revenue]
VAR TestPassed = ABS(ActualResult - ExpectedResult) < 0.01
RETURN 
    IF(
        TestPassed,
        "✅ PASS: Revenue calculation correct",
        "❌ FAIL: Expected " & ExpectedResult & ", Got " & ActualResult
    )

// Test Data Quality
Test_Data_Quality = 
VAR DuplicateOrders = 
    SUMX(
        VALUES(Sales[OrderID]),
        IF(CALCULATE(COUNT(Sales[OrderID])) > 1, 1, 0)
    )
VAR MissingCustomers = 
    COUNTX(Sales, IF(ISBLANK(Sales[CustomerID]), 1, 0))
RETURN
    IF(
        DuplicateOrders = 0 && MissingCustomers = 0,
        "✅ Data Quality OK",
        "❌ Data Issues: " & DuplicateOrders & " duplicates, " & MissingCustomers & " missing customers"
    )
```

### F. Advanced Scenarios

#### Dynamic Security (RLS)
```dax
// User-based filtering
[Security Filter] = 
VAR CurrentUser = USERPRINCIPALNAME()
VAR UserRegion = 
    LOOKUPVALUE(
        UserSecurity[Region],
        UserSecurity[Email], CurrentUser
    )
RETURN Sales[Region] = UserRegion

// Hierarchical Security
[Manager Security] = 
VAR CurrentUser = USERPRINCIPALNAME()
VAR UserLevel = 
    LOOKUPVALUE(
        UserSecurity[Level],
        UserSecurity[Email], CurrentUser
    )
RETURN
    SWITCH(
        UserLevel,
        "CEO", TRUE(),
        "Regional Manager", Sales[Region] = LOOKUPVALUE(UserSecurity[Region], UserSecurity[Email], CurrentUser),
        "Sales Rep", Sales[SalesRep] = CurrentUser,
        FALSE()
    )
```

#### Advanced Time Intelligence
```dax
// Custom Fiscal Year (April to March)
Fiscal_YTD = 
VAR FiscalYearStart = 
    IF(
        MONTH(TODAY()) >= 4,
        DATE(YEAR(TODAY()), 4, 1),
        DATE(YEAR(TODAY()) - 1, 4, 1)
    )
RETURN
    CALCULATE(
        SUM(Sales[Revenue]),
        DATESBETWEEN(
            Dates[Date],
            FiscalYearStart,
            TODAY()
        )
    )

// Weekday vs Weekend Analysis
Weekday_Sales = 
CALCULATE(
    SUM(Sales[Revenue]),
    FILTER(
        ALL(Dates),
        WEEKDAY(Dates[Date], 2) <= 5 // Monday = 1, Sunday = 7
    )
)

Weekend_Sales = 
CALCULATE(
    SUM(Sales[Revenue]),
    FILTER(
        ALL(Dates),
        WEEKDAY(Dates[Date], 2) > 5
    )
)
```

#### Complex Business Logic
```dax
// Tiered Commission Calculation
Sales_Commission = 
VAR SalesAmount = [Total Sales]
VAR Commission = 
    SWITCH(
        TRUE(),
        SalesAmount <= 10000, SalesAmount * 0.02,
        SalesAmount <= 50000, 10000 * 0.02 + (SalesAmount - 10000) * 0.03,
        SalesAmount <= 100000, 10000 * 0.02 + 40000 * 0.03 + (SalesAmount - 50000) * 0.04,
        10000 * 0.02 + 40000 * 0.03 + 50000 * 0.04 + (SalesAmount - 100000) * 0.05
    )
RETURN Commission

// Multi-currency Conversion
Revenue_USD = 
SUMX(
    Sales,
    VAR TransactionDate = Sales[Date]
    VAR TransactionCurrency = Sales[Currency]
    VAR ExchangeRate = 
        LOOKUPVALUE(
            ExchangeRates[Rate],
            ExchangeRates[Date], TransactionDate,
            ExchangeRates[FromCurrency], TransactionCurrency,
            ExchangeRates[ToCurrency], "USD"
        )
    RETURN Sales[Revenue] * COALESCE(ExchangeRate, 1)
)
```

### G. Integration với Power Platform

#### Power Automate Integration
```dax
// Status cho Flow triggers
Approval_Status = 
VAR PendingApprovals = 
    CALCULATE(
        COUNT(Approvals[ID]),
        Approvals[Status] = "Pending"
    )
RETURN
    IF(
        PendingApprovals > 0,
        "Action Required: " & PendingApprovals & " pending approvals",
        "All Clear"
    )
```

#### Power Apps Integration
```dax
// Validate data cho Power Apps
Data_Validation = 
VAR RequiredFields = 
    COUNTX(
        Sales,
        IF(
            OR(
                ISBLANK(Sales[CustomerID]),
                ISBLANK(Sales[ProductID]),
                ISBLANK(Sales[Revenue])
            ),
            1,
            0
        )
    )
RETURN
    IF(
        RequiredFields = 0,
        "Valid",
        RequiredFields & " records missing required fields"
    )
```

### H. Deployment và Version Control

#### Model Documentation
```dax
// Self-documenting measures
Total_Sales = 
/* 
Purpose: Calculate total sales revenue
Author: [Your Name]
Created: [Date]
Last Modified: [Date]
Dependencies: Sales[Revenue]
Business Rules: Includes all revenue, excludes refunds
*/
SUM(Sales[Revenue])

// Version tracking
Model_Version = "v2.1.0 - Updated 2024-12-31"
Last_Refresh = "Data last refreshed: " & FORMAT(NOW(), "DD/MM/YYYY HH:MM")
```

### I. Troubleshooting Guide

#### Common Issues và Solutions

**Issue: Measure returns BLANK**
```dax
// Debug pattern
Debug_Measure = 
VAR RowCount = COUNTROWS(Sales)
VAR SumValue = SUM(Sales[Revenue])
VAR FilterInfo = "Filters: " & CONCATENATEX(FILTERS(Sales), [FilterExpression], "; ")
RETURN
    "Rows: " & RowCount & 
    " | Sum: " & SumValue & 
    " | " & FilterInfo
```

**Issue: Performance Problems**
```dax
// Use query profiling
Optimized_Measure = 
VAR StartTime = NOW()
VAR Result = [Your_Complex_Calculation]
VAR EndTime = NOW()
VAR Duration = DATEDIFF(StartTime, EndTime, SECOND)
RETURN 
    IF(
        SELECTEDVALUE(Debug[ShowTiming], FALSE()),
        "Result: " & Result & " (Took " & Duration & " seconds)",
        Result
    )
```

### J. Future-Proofing Your DAX

#### Preparing for Updates
```dax
// Use semantic layer
Revenue_Base = SUM(Sales[Revenue])
Revenue_Adjusted = [Revenue_Base] * [Adjustment_Factor]

// Flexible date handling
Smart_Date_Filter = 
VAR DateColumn = 
    IF(
        HASONEVALUE(Config[DateColumn]),
        SELECTEDVALUE(Config[DateColumn]),
        "OrderDate"
    )
RETURN
    SWITCH(
        DateColumn,
        "OrderDate", CALCULATE([Total Sales], USERELATIONSHIP(Sales[OrderDate], Dates[Date])),
        "ShipDate", CALCULATE([Total Sales], USERELATIONSHIP(Sales[ShipDate], Dates[Date])),
        [Total Sales]
    )
```

---

## Lời Kết

Tài liệu này cung cấp một cái nhìn toàn diện về DAX trong Power BI, từ những khái niệm cơ bản nhất đến các kỹ thuật nâng cao nhất. DAX là một công cụ mạnh mẽ, nhưng cũng đòi hỏi thời gian và sự thực hành để thành thạo.

### Key Takeaways:
1. **Context is King**: Hiểu Row Context và Filter Context là chìa khóa
2. **Practice Makes Perfect**: Thực hành với dữ liệu thực tế
3. **Performance Matters**: Luôn nghĩ về hiệu suất khi viết DAX
4. **Community Support**: Tận dụng cộng đồng Power BI
5. **Continuous Learning**: DAX liên tục được cập nhật và mở rộng

### Hành Trình Tiếp Theo:
- Thực hành với các scenarios trong tài liệu này
- Tham gia các project thực tế
- Đóng góp vào community
- Chuẩn bị cho Microsoft certifications
- Khám phá Advanced Analytics với Power BI

**Chúc bạn thành công trong hành trình làm chủ DAX và trở thành Power BI Expert!**

---

*Tài liệu này sẽ được cập nhật thường xuyên để phản ánh những thay đổi mới nhất trong Power BI và DAX. Hãy bookmark và quay lại thường xuyên để cập nhật kiến thức!*