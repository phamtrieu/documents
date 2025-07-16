## 📋 TỔNG QUAN HỆ THỐNG

### Mục đích
Hệ thống phân tích chương trình khuyến mại sử dụng phương pháp luận nâng cao để đánh giá hiệu quả, khả thi và rủi ro của các chương trình khuyến mại dựa trên:
- **Dữ liệu lịch sử bán hàng** (3 tháng, 6 tháng, cùng kỳ năm trước)
- **Chỉ số tăng trưởng động** (growth momentum, volatility)
- **Yếu tố thị trường** (seasonality, competition)
- **Phân tích rủi ro đa chiều**

### Phạm vi áp dụng
- Đánh giá trước khi triển khai chương trình khuyến mại
- Tối ưu hóa số lượng và thời điểm khuyến mại
- Dự báo hiệu quả tài chính và rủi ro thị trường

---

## 📊 CẤU TRÚC DỮ LIỆU ĐẦU VÀO

### A. Dữ liệu cơ bản từ DB

#### 1. Thông tin chương trình
- `pro_number`: Mã chương trình khuyến mại
- `areal`: Khu vực áp dụng
- `cost_center`: Phòng kinh doanh
- `from_date`, `to_date`: Thời gian triển khai

#### 2. Thông tin sản phẩm và giá
- `item_code`: Mã sản phẩm
- `promo_qty`: Số lượng khuyến mại
- `promo_price`: Giá bán khuyến mại (không VAT)
- `promo_cost_price`: Giá vốn
- `promo_netprice`: Giá bán ròng
- `expense_price`: Chi phí vận hành + quỹ sử dụng

#### 3. Dữ liệu bán hàng lịch sử
ví dụ: hiện tại tính ctkm cho tháng 7 năm 2025
- `last_3_months_avg_qty`: Số lượng bán trung bình 3 tháng gần nhất (là TB của tháng 5,6,7 năm 2025)
- `last_6_months_avg_qty`: Số lượng bán trung bình 6 tháng gần nhất  (là TB của tháng 2,3,4, 5,6,7 năm 2025)
- `period_last_year_3m_qty`: Số lượng bán cùng kỳ năm trước (là TB của tháng 7,8,9 năm 2024)

#### 4. Dữ liệu cạnh tranh
- `promo_qty_other`: Tổng số lượng khuyến mại các CTKM Khác đã duyệt cùng costcenter, cùng item
- `total_promo_qty`: Tổng số lượng khuyến mại (bao gồm các ctkm khác đã duyệt có cùng costcenter, itemcode và ctkm đang phân tích)

---

## 🔬 CÁC CHỈ SỐ TĂNG TRƯỞNG ĐƯỢC TÍNH TOÁN

### A. Chỉ số tăng trưởng ngắn hạn (Short-term Growth Rate)

**Công thức:**
```
short_term_growth_rate = ((last_3_months_avg_qty - last_6_months_avg_qty) / last_6_months_avg_qty) × 100
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đo lường xu hướng tăng trưởng gần đây nhất, phản ánh momentum hiện tại của sản phẩm
- **Cách hiểu**: So sánh doanh số 3 tháng gần nhất với trung bình 6 tháng gần nhất
- **Giá trị tích cực (>0)**: Sản phẩm đang có xu hướng tăng trưởng → **CAO LÀ TỐT**
- **Giá trị tiêu cực (<0)**: Sản phẩm đang suy giảm → **THẤP LÀ XẤU**

**Ứng dụng thực tế:**
- **Cao (≥20%)**: Sản phẩm "HOT", thị trường đang có nhu cầu mạnh → Tăng số lượng KM
- **Trung bình (5-19%)**: Sản phẩm ổn định, có triển vọng → Duy trì kế hoạch
- **Thấp (0-4%)**: Sản phẩm đang trì trệ → Cần kích thích bằng KM mạnh
- **Âm (<0%)**: Sản phẩm đang "lạnh" → Xem xét giảm quy mô hoặc hủy KM

**Phân loại giá trị:**
- `≥ 20%`: Tăng trưởng rất cao (VERY_HIGH) ✅ **TỐT NHẤT**
- `10% - 19.99%`: Tăng trưởng cao (HIGH) ✅ **TỐT**
- `5% - 9.99%`: Tăng trưởng vừa phải (MODERATE) ⚠️ **KHHA**
- `0% - 4.99%`: Tăng trưởng thấp (LOW) ⚠️ **YẾU**
- `-10% - -0.01%`: Đang suy giảm (DECLINING) ❌ **XẤU**
- `< -10%`: Suy giảm mạnh (STEEP_DECLINE) ❌ **RẤT XẤU**

### B. Chỉ số tăng trưởng cùng kỳ (YoY Growth Rate)

**Công thức:**
```
yoy_growth_rate = ((last_3_months_avg_qty - period_last_year_3m_qty) / period_last_year_3m_qty) × 100
```

**Ý nghĩa chi tiết:**
- **Mục đích**: So sánh hiệu suất với cùng thời điểm năm trước, loại bỏ yếu tố mùa vụ
- **Cách hiểu**: Đánh giá sự phát triển thực sự của sản phẩm qua thời gian dài
- **Giá trị tích cực (>0)**: Sản phẩm tăng trưởng so với năm trước → **CAO LÀ TỐT**
- **Giá trị tiêu cực (<0)**: Sản phẩm suy giảm so với năm trước → **THẤP LÀ XẤU**

**Ứng dụng thực tế:**
- **Cao (≥30%)**: Sản phẩm đang phát triển mạnh mẽ → Đầu tư KM tích cực
- **Tích cực (5-29%)**: Sản phẩm có tăng trưởng bền vững → Duy trì đà phát triển
- **Trung tính (-5% đến 4%)**: Sản phẩm đi ngang → Cần kích thích
- **Tiêu cực (<-5%)**: Sản phẩm đang mất thị phần → Cần chiến lược cứu vãn

**Tầm quan trọng**: Chỉ số này quan trọng hơn short-term growth vì:
- Phản ánh xu hướng dài hạn
- Loại bỏ nhiễu ngắn hạn
- Cơ sở đánh giá chiến lược sản phẩm

**Phân loại giá trị:** (Tương tự short-term growth rate với ý nghĩa như trên)

### C. Chỉ số động lực tăng trưởng (Growth Momentum)

**Công thức logic:**
```
growth_momentum = short_term_growth_rate > yoy_growth_rate ? value > 1.0 : value < 1.0
(Giá trị cụ thể được tính toán dựa trên tỷ lệ gia tốc/chậm lại)
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đánh giá sản phẩm đang tăng tốc hay chậm lại so với xu hướng dài hạn
- **Cách hiểu**: So sánh momentum gần đây với xu hướng cùng kỳ năm trước
- **Giá trị > 1.0**: Đang tăng tốc → **CAO LÀ TỐT**
- **Giá trị < 1.0**: Đang chậm lại → **THẤP LÀ XẤU**

**Ứng dụng thực tế:**
- **Tăng tốc mạnh (≥1.5)**: Sản phẩm đang "bùng nổ" → Thời điểm vàng cho KM quy mô lớn
- **Cải thiện (1.2-1.49)**: Momentum tích cực → Tăng đầu tư marketing
- **Ổn định (1.0-1.19)**: Đang duy trì đà → Giữ kế hoạch hiện tại
- **Chậm lại (0.8-0.99)**: Cần cảnh giác → Xem xét điều chỉnh chiến lược
- **Suy giảm (<0.8)**: Mất momentum → Cần hành động khẩn cấp hoặc tạm dừng KM

**Tầm quan trọng**: Chỉ số dự báo tương lai quan trọng nhất
- Phát hiện sớm xu hướng thay đổi
- Cơ sở timing cho các quyết định KM
- Tránh đầu tư vào sản phẩm đang "hết hơi"

**Phân loại giá trị:**
- `≥ 1.5`: Đang tăng tốc (ACCELERATING) 🚀 **TỐT NHẤT**
- `1.2 - 1.49`: Đang cải thiện (IMPROVING) ✅ **TỐT**
- `1.0 - 1.19`: Ổn định (STABLE) ⚖️ **BÌNH THƯỜNG**
- `0.8 - 0.99`: Đang chậm lại (SLOWING) ⚠️ **CẢNH BÁO**
- `< 0.8`: Đang suy giảm (DECLINING) ❌ **XẤU**

### D. Chỉ số biến động tăng trưởng (Growth Volatility)

**Công thức:**
```
growth_volatility = |short_term_growth_rate - yoy_growth_rate|
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đo độ ổn định của xu hướng tăng trưởng, phát hiện sự bất thường
- **Cách hiểu**: Khoảng cách giữa xu hướng ngắn hạn và dài hạn
- **Giá trị thấp**: Tăng trưởng ổn định, dễ dự báo → **THẤP LÀ TỐT**
- **Giá trị cao**: Tăng trưởng không ổn định, khó dự báo → **CAO LÀ XẤU**

**Ứng dụng thực tế:**
- **Rất ổn định (≤0.2)**: Sản phẩm có xu hướng rõ ràng → An tâm đầu tư KM dài hạn
- **Ổn định (0.21-0.4)**: Biến động nhẹ, vẫn kiểm soát được → Thực hiện KM bình thường
- **Vừa phải (0.41-0.6)**: Có dấu hiệu bất ổn → Cần monitoring sát sao
- **Biến động (0.61-0.8)**: Xu hướng không rõ ràng → Giảm quy mô KM, tăng flexibility
- **Rất biến động (>0.8)**: Thị trường hỗn loạn → Tạm dừng KM hoặc chỉ thực hiện ngắn hạn

**Tầm quan trọng**: Chỉ số đánh giá rủi ro quan trọng
- Phát hiện sản phẩm có vấn đề cấu trúc
- Cảnh báo về tính không chắc chắn
- Cơ sở điều chỉnh quy mô đầu tư

**Nguyên nhân volatility cao:**
- Sản phẩm đang chuyển đổi lifecycle
- Ảnh hưởng của competitor
- Thay đổi hành vi khách hàng
- Yếu tố bên ngoài (kinh tế, chính sách)

**Phân loại giá trị:**
- `≤ 0.2`: Rất ổn định (VERY_STABLE) ✅ **TỐT NHẤT**
- `0.21 - 0.4`: Ổn định (STABLE) ✅ **TỐT**
- `0.41 - 0.6`: Vừa phải (MODERATE) ⚠️ **CẦN THEO DÕI**
- `0.61 - 0.8`: Biến động (VOLATILE) ⚠️ **RỦI RO**
- `> 0.8`: Rất biến động (VERY_VOLATILE) ❌ **RỦI RO CAO**

### E. Chỉ số mùa vụ (Seasonality Index)

**Công thức:**
```
seasonality_index = current_month_avg / annual_avg_monthly
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đánh giá ảnh hưởng của yếu tố mùa vụ đến nhu cầu sản phẩm
- **Cách hiểu**: So sánh doanh số tháng hiện tại với trung bình cả năm
- **Giá trị > 1.0**: Đây là mùa thuận lợi → **CAO LÀ TỐT**
- **Giá trị < 1.0**: Đây là mùa khó khăn → **THẤP LÀ XẤU**

**Ứng dụng thực tế:**
- **Mùa cao điểm (≥1.3)**: Nhu cầu tự nhiên cao → Thời điểm vàng cho KM, tối đa hóa lợi nhuận
- **Mùa thuận lợi (1.1-1.29)**: Nhu cầu tốt → Thực hiện KM bình thường với kỳ vọng cao
- **Mùa bình thường (0.9-1.09)**: Nhu cầu trung tính → KM để tăng thị phần
- **Mùa thấp điểm (0.7-0.89)**: Nhu cầu yếu → KM kích thích hoặc xả kho
- **Ngoài mùa (<0.7)**: Nhu cầu rất thấp → Tránh KM hoặc chỉ thanh lý

**Ví dụ thực tế:**
- **Kem (hè)**: Index ≥ 1.5 → Mùa cao điểm
- **Áo ấm (đông)**: Index ≥ 1.3 → Mùa thuận lợi  
- **Bánh trung thu (tháng 8-9)**: Index ≥ 2.0 → Cực kỳ mùa vụ
- **Sữa bột**: Index ≈ 1.0 → Không mùa vụ

**Tầm quan trọng**: Timing là everything trong KM
- Tránh "ngược mùa" gây lãng phí
- Tối đa hóa hiệu quả khi "thuận mùa"
- Cơ sở lập kế hoạch tồn kho

**Phân loại giá trị:**
- `≥ 1.3`: Mùa cao điểm (PEAK_SEASON) 🌟 **TỐT NHẤT**
- `1.1 - 1.29`: Mùa thuận lợi (HIGH_SEASON) ✅ **TỐT**
- `0.9 - 1.09`: Mùa bình thường (NORMAL_SEASON) ⚖️ **TRUNG TÍNH**
- `0.7 - 0.89`: Mùa thấp điểm (LOW_SEASON) ⚠️ **KHÓ KHĂN**
- `< 0.7`: Ngoài mùa (OFF_SEASON) ❌ **RẤT KHÓ KHĂN**

---

## 📈 CÁC CHỈ SỐ TÀI CHÍNH CƠ BẢN

### A. Tỷ suất lợi nhuận (Profit Margin)

**Công thức:**
```
profit_margin = ((promo_netprice - promo_cost_price) / promo_price) × 100
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đo lường hiệu quả sinh lời của từng đồng doanh thu
- **Cách hiểu**: Sau khi trừ giá vốn, còn lại bao nhiêu % để chi trả chi phí và lợi nhuận
- **Giá trị cao**: Hiệu quả kinh doanh tốt → **CAO LÀ TỐT**
- **Giá trị thấp/âm**: Kinh doanh kém hiệu quả → **THẤP LÀ XẤU**

**Ứng dụng thực tế:**
- **Rất cao (≥30%)**: Sản phẩm "bò sữa" → Tăng đầu tư, mở rộng quy mô
- **Tốt (20-29%)**: Sinh lời tốt → Duy trì và phát triển
- **Khá (15-19%)**: Chấp nhận được → Tìm cách tối ưu chi phí
- **Thấp (10-14%)**: Cần cải thiện → Xem xét lại chiến lược giá
- **Rất thấp (5-9%)**: Nguy hiểm → Cần hành động khẩn cấp
- **Âm (<0%)**: Lỗ → Dừng ngay hoặc tái cấu trúc

**Benchmark ngành:**
- **FMCG**: 15-25% là tốt
- **Điện tử**: 10-20% là bình thường
- **Thời trang**: 40-60% là mục tiêu
- **F&B**: 20-35% là hợp lý

**Ngưỡng đánh giá:**
- `≥ 30%`: Xuất sắc (30 điểm) 🌟 **TỐT NHẤT**
- `25-29.99%`: Rất tốt (26 điểm) ✅ **RẤT TỐT**
- `20-24.99%`: Tốt (22 điểm) ✅ **TỐT**
- `15-19.99%`: Khá (18 điểm) ⚠️ **CHẤP NHẬN**
- `10-14.99%`: Trung bình (12 điểm) ⚠️ **CẦN CẢI THIỆN**
- `5-9.99%`: Thấp (6 điểm) ❌ **YẾU**
- `< 5%`: Rất thấp (0 điểm) ❌ **RẤT YẾU**

### B. Tỷ suất sinh lời (ROI - Return on Investment)

**Công thức:**
```
roi = ((promo_netprice - promo_cost_price - expense_price) / expense_price) × 100
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đo lường hiệu quả của chi phí marketing/quảng cáo
- **Cách hiểu**: Mỗi đồng chi cho KM sẽ tạo ra bao nhiêu đồng lợi nhuận
- **Giá trị cao**: Đầu tư hiệu quả → **CAO LÀ TỐT**
- **Giá trị thấp/âm**: Đầu tư kém hiệu quả → **THẤP LÀ XẤU**

**Ứng dụng thực tế:**
- **Rất cao (≥50%)**: ROI xuất sắc → Tăng ngân sách marketing ngay
- **Tốt (30-49%)**: Hiệu quả tốt → Nhân rộng mô hình
- **Khá (20-29%)**: Chấp nhận được → Tối ưu để tăng hiệu quả
- **Thấp (10-19%)**: Cần xem xét → Điều chỉnh chiến lược
- **Rất thấp (0-9%)**: Gần như vô nghĩa → Cần thay đổi căn bản
- **Âm (<0%)**: Lỗ vốn → Dừng ngay lập tức

**So sánh với các hình thức đầu tư khác:**
- **Gửi ngân hàng**: ~5-8%/năm
- **Chứng khoán**: ~10-15%/năm (trung bình)
- **KM tốt**: >30% trong vài tháng
- **Digital marketing**: 200-400% không hiếm

**Lưu ý quan trọng:**
- ROI KM thường tính trong thời gian ngắn (1-3 tháng)
- Cần tính cả giá trị brand và customer lifetime
- ROI âm có thể chấp nhận nếu để penetrate market

**Ngưỡng đánh giá:**
- `≥ 50%`: Xuất sắc (30 điểm) 🌟 **TỐT NHẤT**
- `30-49.99%`: Rất tốt (25 điểm) ✅ **RẤT TỐT**
- `20-29.99%`: Tốt (20 điểm) ✅ **TỐT**
- `10-19.99%`: Khá (10 điểm) ⚠️ **CHẤP NHẬN**
- `0-9.99%`: Thấp (5 điểm) ⚠️ **YẾU**
- `< 0%`: Lỗ (0 điểm) ❌ **RẤT XẤU**

### C. Tỷ lệ cầu (Demand Ratio)

**Công thức:**
```
demand_ratio = promo_qty / last_3_months_avg_qty
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đánh giá tính hợp lý của số lượng khuyến mại so với nhu cầu thực tế
- **Cách hiểu**: Số lượng KM bằng bao nhiêu lần doanh số bình thường
- **Giá trị hợp lý (1-2)**: Phù hợp với nhu cầu → **VỪA PHẢI LÀ TỐT**
- **Giá trị quá cao (>3)**: Rủi ro thanh khoản → **CAO LÀ XẤU**
- **Giá trị quá thấp (<0.8)**: Bỏ lỡ cơ hội → **THẤP LÀ XẤU**

**Ứng dụng thực tế:**
- **Lý tưởng (1.0-2.0)**: Đúng nhu cầu → Tỷ lệ success cao, ít tồn kho
- **Hơi thấp (0.8-0.99)**: Thận trọng → An toàn nhưng có thể bỏ lỡ doanh thu
- **Hơi cao (2.01-3.0)**: Tham vọng → Cần kế hoạch thanh lý dự phòng
- **Quá thấp (0.5-0.79)**: Quá bảo thủ → Mất cơ hội, competitor có thể chiếm chỗ
- **Quá cao (3.01-5.0)**: Rủi ro cao → Nguy cơ tồn kho, cash flow âm
- **Cực cao (>5.0)**: Rất nguy hiểm → Có thể phá sản nếu không bán được

**Yếu tố cần xem xét:**
- **Sản phẩm mới**: Có thể chấp nhận ratio cao để penetrate
- **Sản phẩm mature**: Nên giữ ratio thấp, ổn định
- **Mùa cao điểm**: Có thể tăng ratio lên 2-3
- **Ngoài mùa**: Nên giữ ratio <1.5

**Rủi ro khi ratio cao:**
- Tồn kho lâu → hàng hết date, mất giá
- Chiếm dụng vốn → ảnh hưởng cash flow
- Tâm lý "hàng ế" → khó bán với giá tốt

**Ngưỡng đánh giá:**
- `1.0 - 2.0`: Hợp lý (25 điểm) 🎯 **TỐT NHẤT**
- `0.8 - 0.99`: Khá hợp lý (22 điểm) ✅ **TỐT**
- `2.01 - 3.0`: Hơi cao (20 điểm) ⚠️ **CHẤP NHẬN**
- `0.5 - 0.79`: Thấp (15 điểm) ⚠️ **BỎ LỠ CƠ HỘI**
- `3.01 - 5.0`: Cao (10 điểm) ❌ **RỦI RO**
- `> 5.0`: Rất cao - rủi ro (3 điểm) ❌ **RẤT NGUY HIỂM**

### D. Tỷ lệ cạnh tranh (Competition Ratio)

**Công thức:**
```
competition_ratio = promo_qty_other / total_promo_qty
```

**Ý nghĩa chi tiết:**
- **Mục đích**: Đánh giá mức độ cạnh tranh trong cùng thời điểm khuyến mại
- **Cách hiểu**: Tỷ lệ % khuyến mại của đối thủ so với tổng thị trường
- **Giá trị thấp**: Ít cạnh tranh, dễ thành công → **THẤP LÀ TỐT**
- **Giá trị cao**: Cạnh tranh gay gắt, khó nổi bật → **CAO LÀ XẤU**

**Ứng dụng thực tế:**
- **Cạnh tranh thấp (≤0.3)**: "Blue Ocean" → Cơ hội vàng, có thể tăng giá hoặc margin
- **Cạnh tranh vừa (0.31-0.5)**: Thị trường bình thường → Cần differentiation rõ ràng
- **Cạnh tranh cao (0.51-0.7)**: "Red Ocean" → Cần USP mạnh, có thể phải giảm margin
- **Cạnh tranh rất cao (>0.7)**: "Blood Bath" → Tránh hoặc có chiến lược đột phá

**Chiến lược theo mức độ cạnh tranh:**

**Khi cạnh tranh thấp:**
- Tăng số lượng, chiếm market share
- Giữ margin cao
- Build brand awareness

**Khi cạnh tranh cao:**
- Focus vào target segment cụ thể
- Nhấn mạnh differentiation
- Có thể sacrifice margin để gain volume

**Yếu tố ảnh hưởng:**
- **Timing**: Cùng lúc nhiều brand làm KM
- **Seasonality**: Mùa cao điểm thường cạnh tranh cao
- **Product lifecycle**: Sản phẩm mature cạnh tranh cao hơn
- **Market maturity**: Thị trường bão hòa → cạnh tranh cao

**Ngưỡng đánh giá:**
- `≤ 0.3`: Cạnh tranh thấp (20 điểm) 🌟 **TỐT NHẤT**
- `0.31 - 0.5`: Cạnh tranh vừa (15 điểm) ✅ **CHẤP NHẬN**
- `0.51 - 0.7`: Cạnh tranh cao (8 điểm) ⚠️ **KHÓ KHĂN**
- `> 0.7`: Cạnh tranh rất cao (0 điểm) ❌ **RẤT KHÓ KHĂN**

---

## 🎯 HỆ THỐNG ĐIỂM SỐ NÂNG CAO

### A. Điểm khả thi nâng cao (Enhanced Feasibility Score) - 100 điểm

**Cấu trúc điểm:**
1. **Đánh giá nhu cầu cơ bản** (25 điểm): Dựa trên demand_ratio
2. **Đánh giá cạnh tranh** (20 điểm): Dựa trên competition_ratio  
3. **Đánh giá thời gian** (15 điểm): Dựa trên thời gian chương trình
4. **Đánh giá số lượng hợp lý** (10 điểm): Tính hợp lý của promo_qty
5. **Đánh giá tăng trưởng ngắn hạn** (10 điểm): Dựa trên short_term_growth_rate
6. **Đánh giá tăng trưởng cùng kỳ** (8 điểm): Dựa trên yoy_growth_rate
7. **Đánh giá xu hướng tăng trưởng** (7 điểm): Dựa trên growth_momentum
8. **Đánh giá độ ổn định** (5 điểm): Dựa trên growth_volatility

### B. Điểm lợi nhuận nâng cao (Enhanced Profitability Score) - 100 điểm

**Cấu trúc điểm:**
1. **Profit Margin cơ bản** (30 điểm)
2. **ROI cơ bản** (30 điểm)
3. **Growth-Adjusted Profitability** (25 điểm): Lợi nhuận điều chỉnh theo tăng trưởng
4. **Risk-Adjusted Return** (15 điểm): Return điều chỉnh theo rủi ro

### C. Điểm thời điểm thị trường (Market Timing Score) - 100 điểm

**Cấu trúc điểm:**
1. **Đánh giá theo mùa vụ** (30 điểm): Dựa trên seasonality_index
2. **Đánh giá momentum thị trường** (40 điểm): Kết hợp short_term và yoy growth
3. **Đánh giá xu hướng** (30 điểm): Dựa trên growth_momentum

### D. Điểm rủi ro tăng trưởng (Growth Risk Score) - 100 điểm

**Cấu trúc điểm:**
1. **Xu hướng tăng trưởng tổng thể** (40 điểm)
2. **Momentum** (30 điểm)  
3. **Độ ổn định** (30 điểm)

### E. Điểm tổng thể (Overall Score)

**Công thức:**
```
overall_score = (enhanced_feasibility_score × 0.35) + 
                (enhanced_profitability_score × 0.25) + 
                (market_timing_score × 0.25) + 
                (growth_risk_score × 0.15)
```

**Điều chỉnh theo anomalies:**
- Trừ 15 điểm cho mỗi anomaly mức độ HIGH
- Trừ 5 điểm cho mỗi anomaly mức độ MEDIUM

---

## ⚠️ HỆ THỐNG PHÁT HIỆN BẤT THƯỜNG (ANOMALY DETECTION)

### A. Bất thường cơ bản (Mức độ nghiêm trọng)

#### 1. **PRICING_ERROR** - Mức độ: HIGH ❌
**Điều kiện**: `promo_price ≤ promo_cost_price`
**Ý nghĩa**: Bán lỗ ngay từ giá vốn
**Tác động**: Chắc chắn lỗ, có thể do nhập sai dữ liệu
**Hành động**: Dừng ngay, kiểm tra lại giá

#### 2. **NEGATIVE_PROFIT** - Mức độ: HIGH ❌
**Điều kiện**: `profit_margin < 0`
**Ý nghĩa**: Lỗ sau khi tính giá vốn
**Tác động**: Càng bán càng lỗ
**Hành động**: Xem xét lại toàn bộ cost structure

#### 3. **NEGATIVE_ROI** - Mức độ: HIGH ❌
**Điều kiện**: `roi < 0`
**Ý nghĩa**: Chi phí marketing > lợi nhuận gross
**Tác động**: Đầu tư vô hiệu quả
**Hành động**: Cắt giảm chi phí marketing hoặc tăng hiệu quả

#### 4. **EXTREME_QUANTITY** - Mức độ: HIGH ❌
**Điều kiện**: `demand_ratio > 10`
**Ý nghĩa**: Số lượng KM gấp >10 lần bán thường
**Tác động**: Rủi ro thanh khoản cực cao, có thể phá sản
**Hành động**: Giảm số lượng xuống mức an toàn

#### 5. **VERY_HIGH_QUANTITY** - Mức độ: HIGH ⚠️
**Điều kiện**: `demand_ratio > 5`
**Ý nghĩa**: Số lượng KM gấp >5 lần bán thường
**Tác động**: Rủi ro tồn kho cao
**Hành động**: Chuẩn bị kế hoạch thanh lý dự phòng

### B. Bất thường liên quan tăng trưởng (Mức độ rủi ro)

#### 6. **DECLINING_TREND** - Mức độ: HIGH ⚠️
**Điều kiện**: `short_term_growth_rate < -20%`
**Ý nghĩa**: Sản phẩm đang suy giảm mạnh
**Tác động**: KM có thể không hiệu quả, thị trường đang bỏ rơi sản phẩm
**Hành động**: Giảm quy mô KM, tìm hiểu nguyên nhân suy giảm

#### 7. **HIGH_VOLATILITY** - Mức độ: MEDIUM ⚠️
**Điều kiện**: `growth_volatility > 0.8`
**Ý nghĩa**: Xu hướng không ổn định, khó dự báo
**Tác động**: Kết quả KM khó đoán trước
**Hành động**: Tăng cường monitoring, giảm commitment dài hạn

#### 8. **TREND_CONFLICT** - Mức độ: MEDIUM ⚠️
**Điều kiện**: `|short_term_growth - yoy_growth| > 30%`
**Ý nghĩa**: Mâu thuẫn giữa xu hướng ngắn hạn và dài hạn
**Tác động**: Khó xác định xu hướng thực sự
**Hành động**: Phân tích sâu hơn về nguyên nhân, thận trọng trong dự báo

#### 9. **POOR_TIMING** - Mức độ: HIGH ❌
**Điều kiện**: `growth_momentum < 0.8 AND demand_ratio > 3`
**Ý nghĩa**: Sản phẩm đang mất momentum nhưng KM với số lượng cao
**Tác động**: Rủi ro kép: sản phẩm yếu + số lượng cao
**Hành động**: Dời lịch KM hoặc giảm quy mô đáng kể

### C. Tác động của anomalies lên điểm số

#### Hệ số penalty:
- **HIGH severity**: -15 điểm/anomaly
- **MEDIUM severity**: -5 điểm/anomaly

#### Logic xử lý:
```
final_score = overall_score - (high_anomalies × 15) - (medium_anomalies × 5)
final_score = max(final_score, 0)  // Không cho điểm âm
```

#### Tác động lên khuyến nghị:
- **≥2 HIGH anomalies**: Tự động hạ xuống "VERY_POOR"
- **≥1 HIGH anomaly**: Không thể đạt "EXCELLENT"
- **≥3 MEDIUM anomalies**: Tương đương 1 HIGH anomaly

---

## 🎯 HỆ THỐNG KHUYẾN NGHỊ CHI TIẾT

### A. Phân loại theo điểm số (Overall Score)

#### **EXCELLENT** (≥85 điểm, 0 anomaly HIGH) 🌟
**Ý nghĩa**: Chương trình xuất sắc, tất cả chỉ số đều tốt
**Đặc điểm**:
- Profit margin ≥ 25%
- ROI ≥ 40% 
- Demand ratio 1.0-2.5
- Growth momentum > 1.2
- Ít hoặc không có rủi ro

**Khuyến nghị hành động**:
- ✅ Thực hiện ngay lập tức
- ✅ Có thể tăng ngân sách marketing
- ✅ Nhân rộng mô hình cho sản phẩm khác
- ✅ Set làm benchmark cho tương lai

#### **VERY_GOOD** (75-84 điểm, 0 anomaly HIGH) ✅
**Ý nghĩa**: Chương trình rất tốt, đa số chỉ số tích cực
**Đặc điểm**:
- Profit margin ≥ 20%
- ROI ≥ 25%
- Các chỉ số tăng trưởng tích cực
- Rủi ro thấp đến trung bình

**Khuyến nghị hành động**:
- ✅ Thực hiện theo kế hoạch
- ✅ Monitor closely để tối ưu
- ⚠️ Sẵn sàng scale up nếu kết quả tốt

#### **GOOD** (65-74 điểm) 👍
**Ý nghĩa**: Chương trình tốt, có một số điểm cần cải thiện
**Đặc điểm**:
- Profit margin 15-25%
- ROI 15-30%
- Một số chỉ số chưa tối ưu
- Có thể có 1-2 anomaly MEDIUM

**Khuyến nghị hành động**:
- ✅ Thực hiện nhưng theo dõi sát
- ⚠️ Tối ưu những điểm yếu đã phát hiện
- ⚠️ Chuẩn bị plan B nếu cần

#### **FAIR** (55-64 điểm) ⚠️
**Ý nghĩa**: Chương trình khá, cần theo dõi chặt chẽ
**Đặc điểm**:
- Profit margin 10-20%
- ROI 10-25%
- Một số rủi ro đã được xác định
- Có thể có anomaly HIGH hoặc nhiều MEDIUM

**Khuyến nghị hành động**:
- ⚠️ Cân nhắc kỹ trước khi thực hiện
- ⚠️ Giảm quy mô để test market
- ⚠️ Tăng cường monitoring real-time
- ⚠️ Chuẩn bị exit strategy

#### **POOR** (45-54 điểm) ❌
**Ý nghĩa**: Chương trình kém, cần cải thiện đáng kể
**Đặc điểm**:
- Profit margin < 15%
- ROI < 20%
- Nhiều chỉ số tiêu cực
- Có anomaly HIGH

**Khuyến nghị hành động**:
- ❌ Không nên thực hiện theo kế hoạch hiện tại
- 🔄 Tái thiết kế chương trình
- 🔄 Xem xét thay đổi sản phẩm/timing/giá
- 🔄 Chỉ test với quy mô nhỏ

#### **VERY_POOR** (<45 điểm) ❌❌
**Ý nghĩa**: Chương trình rất kém, nguy cơ thất bại cao
**Đặc điểm**:
- Profit margin có thể âm
- ROI thấp hoặc âm
- Nhiều anomaly HIGH
- Rủi ro rất cao

**Khuyến nghị hành động**:
- ❌ Không thực hiện
- 🔄 Quay lại drawing board
- 🔄 Phân tích lại toàn bộ giả định
- 🔄 Có thể hủy bỏ hoàn toàn

### B. Điều chỉnh theo yếu tố tăng trưởng

#### **Nâng cấp recommendation** khi:
- `growth_avg > 20%` AND `momentum > 1.3`
- Thị trường đang "nóng" → Tăng tham vọng

**Ví dụ**: GOOD → VERY_GOOD nếu growth factors rất tốt

#### **Hạ cấp recommendation** khi:
- `growth_avg < -10%` OR `momentum < 0.7`
- Thị trường đang "lạnh" → Giảm risk exposure

**Ví dụ**: EXCELLENT → VERY_GOOD nếu growth factors xấu

### C. Cảnh báo đặc biệt

#### 🚨 **Cảnh báo số lượng quá cao**
**Khi**: `demand_ratio > 5`
**Message**: "CẢNH BÁO: Số lượng quá cao!"
**Tác động**: Tự động chuyển risk level lên HIGH
**Action**: Giảm số lượng hoặc chuẩn bị kế hoạch thanh lý

#### ⏰ **Cảnh báo timing không phù hợp**
**Khi**: `momentum < 0.8` AND `demand_ratio > 3`
**Message**: "CẢNH BÁO: Thời điểm không phù hợp!"
**Tác động**: Tự động chuyển risk level lên HIGH
**Action**: Dời lịch hoặc giảm quy mô đáng kể

### D. Ma trận quyết định

| Overall Score | Anomaly HIGH | Anomaly MEDIUM | Final Recommendation |
|---------------|--------------|----------------|---------------------|
| ≥85 | 0 | 0-2 | EXCELLENT ✅ |
| ≥85 | ≥1 | Any | VERY_GOOD ⚠️ |
| 75-84 | 0 | 0-3 | VERY_GOOD ✅ |
| 75-84 | ≥1 | Any | GOOD ⚠️ |
| 65-74 | 0-1 | 0-4 | GOOD 👍 |
| 65-74 | ≥2 | Any | FAIR ⚠️ |
| 55-64 | Any | Any | FAIR ⚠️ |
| 45-54 | Any | Any | POOR ❌ |
| <45 | Any | Any | VERY_POOR ❌❌ |

---

## 📊 PHÂN TÍCH RỦI RO ĐA CHIỀU

### A. Rủi ro thị trường (Market Risk)

**Yếu tố đánh giá:**
- Demand ratio (rủi ro thanh khoản)
- Growth momentum (rủi ro xu hướng)
- YoY growth rate (rủi ro cơ cấu)

**Mức độ rủi ro:**
- **LOW**: Demand ratio ≤ 2, momentum > 1.3, YoY > 15%
- **MEDIUM**: Điều kiện trung bình
- **HIGH**: Demand ratio > 3, momentum < 0.8, YoY < -10%
- **VERY_HIGH**: Demand ratio > 5, điều kiện xấu

### B. Rủi ro thời điểm (Timing Risk)

**Yếu tố đánh giá:**
- Seasonality index
- Growth momentum  
- Short-term growth

**Tính điểm:**
- Seasonality ≥ 1.2: +2 điểm
- Momentum ≥ 1.2: +2 điểm
- Short-term growth ≥ 10%: +2 điểm

**Phân loại:**
- **LOW**: ≥5 điểm
- **MEDIUM**: 3-4 điểm
- **HIGH**: 1-2 điểm
- **VERY_HIGH**: 0 điểm

---

## 📋 PHÂN TÍCH SỨC KHỎE TĂNG TRƯỞNG

### A. Chỉ số sức khỏe tổng thể

**Yếu tố tính toán:**
- Short-term growth > 5%: +1 điểm
- YoY growth > 0%: +1 điểm  
- Growth momentum > 1.0: +1 điểm
- Growth volatility < 0.5: +1 điểm

**Phân loại:**
- **EXCELLENT**: 80-100% yếu tố tích cực
- **GOOD**: 60-79% yếu tố tích cực
- **FAIR**: 40-59% yếu tố tích cực  
- **POOR**: <40% yếu tố tích cực

---

## 🔄 QUY TRÌNH PHÂN TÍCH

### Bước 1: Thu thập dữ liệu 
- Lấy dữ liệu lịch sử 3, 6, 12 tháng
- Tính toán các chỉ số tăng trưởng

### Bước 2: Tính toán chỉ số cơ bản
- Profit margin, ROI
- Demand ratio, Competition ratio

### Bước 3: Phân tích nâng cao
- 4 điểm số chính (Feasibility, Profitability, Timing, Risk)
- Phát hiện anomalies
- Phân tích sức khỏe tăng trưởng

### Bước 4: Tổng hợp và khuyến nghị
- Tính overall score
- Xác định mức độ rủi ro
- Đưa ra khuyến nghị chiến lược

---

## 📈 KẾT QUẢ PHÂN TÍCH

### A. Báo cáo tổng quan
- Thông tin chương trình
- Hiệu quả tài chính tổng thể
- Phân tích tăng trưởng và xu hướng
- Top performers và bottom performers

### B. Dữ liệu chi tiết  
- Điểm số từng SKU
- Phân tích anomalies
- Khuyến nghị cụ thể
- Đánh giá rủi ro

### C. Insights chiến lược
- Tâm lý thị trường tổng thể
- Khuyến nghị về thời điểm và quy mô
- Cảnh báo rủi ro và cơ hội

---

## 🎯 LỢI ÍCH CỦA PHƯƠNG PHÁP LUẬN

### 1. Tính khoa học cao
- Dựa trên dữ liệu lịch sử thực tế
- Kết hợp nhiều chiều đánh giá
- Thuật toán điểm số minh bạch

### 2. Phân tích đa chiều
- Tài chính + Thị trường + Tăng trưởng + Rủi ro
- Phát hiện anomalies tự động
- Đánh giá timing chính xác

### 3. Khuyến nghị thực tế
- Phân loại rõ ràng theo mức độ hiệu quả
- Cảnh báo rủi ro cụ thể
- Khuyến nghị hành động chi tiết

### 4. Hỗ trợ ra quyết định
- Điểm số tổng thể dễ hiểu
- So sánh được giữa các chương trình
- Tối ưu hóa portfolio khuyến mại

---

*Tài liệu này được xây dựng dựa trên Enhanced Promotional Analytics System - Version 2.0 bởi Sunhouse dev team*
*Cập nhật lần cuối: Tháng 7, 2025*
