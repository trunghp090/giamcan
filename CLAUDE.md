# GiảmCân.AI — App theo dõi cân nặng, calo, thực đơn & lịch tập

App web/PWA tiếng Việt, tự chứa trong MỘT file `index.html` (không framework, không build tool, không backend). Chủ dự án: trunghp090 — đồng thời là người dùng chính của app.

## Deploy

- **Bản chính thức:** https://trunghp090.github.io/giamcan/ — GitHub Pages tự deploy từ nhánh `main` của repo này. Push lên `main` là app cập nhật (~1 phút).
- App là PWA: `manifest.webmanifest` + `sw.js` (network-first, fallback cache) + icon PNG. User cài lên điện thoại qua "Thêm vào màn hình chính".
- Có dự án bọc native tại repo `giamcan-mobile` (Capacitor, Android + iOS) — khi sửa `index.html` ở đây, chạy `update-app.sh` bên đó để đồng bộ.

## Cấu trúc index.html

Một file duy nhất: `<head>` (PWA meta) → `<body>` chứa `<title>`, `<style>`, markup, `<script>` chính (IIFE), và script đăng ký service worker cuối body. Khi kiểm tra cú pháp JS phải tách TỪNG khối `<script>` riêng (có 2 khối).

### Lưu trữ (localStorage, có fallback in-memory)
- `gc_profile` — hồ sơ: sex, yob, h, w0, goal (kg mục tiêu), rate (kg/tuần), tgoal (cut/bulk/endurance), place (gym/home), equip {barbell,dumbbell,pullup,treadmill,bike}, foods {trung,sandwich,com,ga,bo,lon,tom,ca,whey,suaHP,suaCD,scVNM,chuoi,khoai,yen}
- `gc_weights` [{d,kg}] · `gc_foods` [{d,name,kcal,qty}] · `gc_workouts` [{d,key,kcal,mins?,hr?,actual?,est?}] · `gc_treats` [ngày ăn ngoài] · `gc_customfoods` [[tên,kcal]] · `gc_menuoff` (xoay thực đơn) · `gc_ai` {prov,key} — key AI KHÔNG đưa vào file xuất dữ liệu

### Các engine chính (đều rule-based, KHÔNG dùng LLM cho tính toán)
- **BMR** Mifflin-St Jeor; **TDEE ước tính** = BMR×1,2 + calo lịch tập/7 (đã bỏ khái niệm "mức vận động")
- **TDEE thích ứng** `adaptiveTDEE(endD)`: cửa sổ 21 ngày, cần ≥8 ngày ghi món + ≥6 lần cân; 7.700 kcal/kg; ngày treat ước lượng = trung bình + 600 kcal; pha trộn với công thức theo độ tin cậy
- **calTarget(d)** — mục tiêu calo tính lại TỪNG NGÀY theo dữ liệu đến ngày đó; sàn an toàn 1500 nam/1200 nữ
- **weekPlan()** — lịch tập 7 ngày sinh từ tgoal + equip (3 template × 3 mức thiết bị: thanh đòn → tạ đơn → trọng lượng cơ thể). ƯU TIÊN THANH ĐÒN (user ghét tạ đơn tháo lắp); HIIT ưu tiên máy chạy hơn xe đạp. Vùng nhịp tim Tanaka 208−0,7×tuổi
- **buildMenu()** — thực đơn ngày CHỈ từ món user tick trong hồ sơ; đạm xoay vòng; ăn phụ trước/sau tập theo lịch; tự lấp calo khớp mục tiêu
- **buildAdvice() + renderBriefing()** — bản tin sáng (thẻ đầu Tổng quan): cân → mục tiêu calo → buổi tập → thực đơn → lời khuyên từ hôm qua. Nhập cân hôm nay xong tự nhảy về bản tin
- **AI nhập món**: gọi thẳng OpenAI gpt-4o-mini hoặc Gemini 2.0 Flash từ trình duyệt, trả JSON {items:[{name,kcal,qty}]}
- **hrKcal(mins,hr)** — ước calo từ nhịp tim (Karvonen) khi user không nhập kcal đồng hồ; kém chính xác hơn số đồng hồ (~±30%)

## Bối cảnh người dùng (quan trọng khi sửa)
- Mục tiêu: giữ cơ + giảm mỡ, kèm sức bền. Tập gym: giá squat, thanh đòn + tạ sàn, ghế ngang, khung xà, máy chạy (KHÔNG dùng máy đạp xe dù gym có)
- Đồng hồ Huawei Watch GT 6 Pro (Huawei Health không có API công khai → nhập tay 3 số phút/kcal/tim TB sau buổi tập)
- Đồ ăn quen: trứng, bánh mì sandwich, cơm, gà/bò/lợn, whey, sữa cao đạm Vinamilk, sữa Vinamilk có đường, sữa chua Vinamilk, chuối
- Nguyên tắc: calo tập luyện KHÔNG cộng vào ngân sách ăn (TDEE thích ứng đã hấp thụ — tránh đếm trùng)

## Việc còn mở
- Thương mại hóa: kế hoạch freemium 39k/tháng · 299k/năm; bước kế tiếp là Firebase Auth + Firestore sync khi có ~20 người dùng
- App store: dự án `giamcan-mobile` đã dựng; user chưa cài Android Studio/Xcode, chưa có tài khoản Play/Apple
- Ý tưởng treo: nhắc xuất backup định kỳ, thông báo đẩy (cần thêm tính năng native khi nộp Apple)
