# QUY TRÌNH SẢN XUẤT VIDEO KINETIC VINFAST "CHẠM TƯƠNG LAI" (1080p 60FPS)

> **Tài liệu hướng dẫn & Bóc tách kỹ thuật toàn diện**  
> Dự án: MV Kinetic "VinFast Chạm Tương Lai"  
> Thời lượng: 38.4 giây | Tỉ lệ: 16:9 (1920x1080) | Tốc độ: 60 FPS  
> Công nghệ lõi: **HTML5 + GSAP 3 + HyperFrames Engine + CTC Forced Alignment + FFmpeg VP9**

---

## 1. TỔNG QUAN LUỒNG SẢN XUẤT (PIPELINE)

```
[ Âm thanh & Lời bài hát ]
          │
          ▼
1. Phân tích CTC Forced Alignment (Torchaudio MMS_FA) ──► Tạo mốc thời gian 78 từ chính xác tới mili-giây
          │
          ▼
2. Thu thập & Chuẩn hóa Assets (VinFast Auto + YouTube TVC) ──► Tách nền xe alpha + scale tỉ lệ CAD thật + encode VP9 WebM
          │
          ▼
3. Xây dựng Kiến trúc Kinetic Code (HTML/CSS + GSAP Timeline) ──► Layout bất đối xứng, Easing expo.out, V-chevron Wipe
          │
          ▼
4. Color Grade Điện ảnh & Nhận diện Thương hiệu ──► Mulish Font, #1464F4 (Blue) + #EE3D43 (Red), Anamorphic Flare
          │
          ▼
5. Render Frame-by-Frame 60FPS (HyperFrames) ──► Xuất file MP4 hoàn chỉnh 1080p60 có âm thanh
```

![Tổng quan quy trình](assets/tong_quan_quy_trinh_contact_sheet.jpg)

---

## 2. CHI TIẾT TỪNG BƯỚC THỰC HIỆN

### BƯỚC 1: XỬ LÝ ÂM THANH & FORCED ALIGNMENT (KHỚP TỪ ĐỘNG)
- **Vấn đề gặp phải:** Whisper API hay Gemini thường bị "hallucinate" hoặc chia timestamp đều nhau (evenly spaced), dẫn đến chữ không giật đúng lúc ca sĩ nhả chữ staccato.
- **Giải pháp kỹ thuật:** 
  - Chuyển đổi audio sang `.wav` chuẩn 16kHz mono.
  - Sử dụng mô hình nhận diện giọng nói **Torchaudio MMS_FA (CTC Forced Alignment)** trên lời bài hát đã phiên âm.
  - Trích xuất mảng `WT` gồm **78 mốc thời gian bắt đầu của từng từ** với độ trễ bù trừ `LEAD = 0.04s` (chữ xuất hiện ngay trước khi âm thanh chạm đỉnh để mắt người cảm nhận khớp hoàn hảo).
  - Tích hợp nhịp nền: Grid 136 BPM (`BEAT = 60/136 = 0.441s`).

---

### BƯỚC 2: CHUẨN BỊ TÀI NGUYÊN (ASSET PIPELINE) & TỶ LỆ XE THẬT
- **Thu thập tài nguyên chính hãng:**
  - Cào dữ liệu ảnh góc chụp ngang (side-view) từ website chính thức `vinfastauto.com`.
  - Tải các đoạn TVC chính thức độ phân giải cao từ kênh VinFast trên YouTube.
- **Xử lý cắt viền alpha (Tight Crop) & Tỉ lệ kích thước thật:**
  - Viền trong suốt không đều khiến xe bị lệch kích thước mắt nhìn. Kỹ thuật viên dùng thuật toán tìm bounding box alpha (`A > 200`) để crop sát từng thân xe.
  - Cố định tỉ lệ chiều dài thực tế ngoài đời:
    - **VF 3:** 3.19 m (nhỏ nhất)
    - **VF 5:** 3.97 m
    - **VF 6:** 4.24 m
    - **VF 7:** 4.55 m
    - **VF 8:** 4.75 m
    - **VF 9:** 5.12 m (lớn nhất)
- **Encode video động "Seek-Safe" (WebM VP9):**
  - Trình duyệt headless khi render frame-by-frame không thể decode H.264 MP4 trơn tru nếu tua liên tục.
  - Toàn bộ video cắt cảnh được chuyển đổi qua `libvpx-vp9` với thông số keyframe định kỳ 15 frame:  
    `ffmpeg -ss [start] -t [dur] -i raw.mp4 -vf "fps=30,scale=1280:720,setsar=1,format=yuv420p,tpad=stop_mode=clone:stop_duration=0.5" -c:v libvpx-vp9 -b:v 0 -crf 28 -g 15 -keyint_min 15 -an clip.webm`

---

### BƯỚC 3: DỰNG CẤU TRÚC 11 PHÂN CẢNH (SCENE BREAKDOWN)

#### Phân đoạn 1: Intro Khởi động (0.0s – 1.2s)
- Biểu tượng cánh chim V chrome bóng bẩy bung nở giữa quầng sáng xanh VinFast và lưới phối cảnh 3D di chuyển về phía trước.
![01 Intro Chrome V](assets/01_intro_chrome_v.jpg)

#### Phân đoạn 2: Count-in Đếm số Staccato (1.2s – 4.8s)
- Lời hát: *“Ba! Năm! Sáu, Bảy! Tám! Tám! Chín, Chín, Chín!”*
- Số typography khổng lồ (size 760px) giật nảy bên trái.
- Các mẫu xe tương ứng (VF 3, VF 5, VF 6, VF 7, VF 8, VF 9) lao vào trên cùng một đường sàn ánh sáng `GROUND = 845px`, kèm bóng phản chiếu gương và vệt bóng mờ tốc độ (motion ghosts).
![02 Count-in VF3](assets/02_countin_vf3.jpg)
![03 Count-in VF9](assets/03_countin_vf9.jpg)

#### Phân đoạn 3: Cú nện Slam Wordmark VINFAST (4.8s – 5.5s)
- Biểu tượng V và Logo chữ VINFAST va đập trực diện vào tâm màn hình với lực quán tính cực mạnh, flash chớp sáng kích hoạt cụm HUD model chips góc dưới.
![04 Slam VinFast](assets/04_slam_vinfast.jpg)

#### Phân đoạn 4: Build-up Giới thiệu từng dòng xe (5.5s – 12.6s)
- Lời hát:
  - *VF 3! Phố đông luồn lách!*
  - *VF 5! Chạy nhanh bất chấp!*
  - *VF 6! Đẹp từng góc cạnh!*
  - *VF 7! Phi cơ cất cánh!*
- Khung hình vát xéo polygon mang DNA chữ V.
- Tích hợp video TVC chuyển động thực tế cho từng dòng xe bên phải, typography kinetic dập nảy bên trái.
![05 Buildup VF3](assets/05_buildup_vf3_tvc.jpg)
![06 Buildup VF7](assets/06_buildup_vf7_tvc.jpg)

#### Phân đoạn 5: Drop Năng lượng cao (12.6s – 17.8s)
- Lời hát: *“Bật nguồn! Cắm sạc! Vút êm! Đạp ga! MỘT! HAI! BA! BỨT PHÁ!”*
- **Bật nguồn:** Cảnh quay thao tác bấm phím điều khiển khởi động thực tế kết hợp vòng power phát sáng.
- **Cắm sạc:** Video thao tác cắm súng sạc vào cổng sạc xe điện, pin ảo nạp đầy 100%.
- **Vút êm & Đạp ga:** Video xe bứt tốc trên đèo kết hợp đồng hồ tốc độ quét góc cung tròn và speedlines.
- **Đếm nhịp 1 - 2 - 3 BỨT PHÁ:** Cắt nhịp dồn dập vào đầu xe các dòng, kết thúc bằng cú va đập phá vỡ góc nhìn qua VF 9.
![07 Bật nguồn](assets/07_drop_bat_nguon.jpg)
![08 Cắm sạc](assets/08_drop_cam_sac.jpg)
![09 Vút êm](assets/09_drop_vut_em.jpg)
![10 Bứt phá](assets/10_drop_but_pha.jpg)

#### Phân đoạn 6: Riser Đội hình hội tụ (17.8s – 20.8s)
- Đoạn synth không lời dâng trào năng lượng: Toàn bộ 6 mẫu xe xếp thành 2 hàng tách bạch, di chuyển tiến công về phía trước, không đè lấn nhau.
![11 Đội hình hội tụ](assets/11_riser_convoy.jpg)

#### Phân đoạn 7: Chorus Điệp khúc uy lực (20.8s – 27.9s)
- Lời hát: *“VF TÁM! Vững chãi đường dài! VF CHÍN! Đẳng cấp tương lai! Lướt êm ru! Không khói bụi! Bật chế độ xanh, VinFast dẫn đầu!”*
- VF 8 và VF 9 xuất hiện trên cung đường biển hùng vĩ.
- Cảnh tượng không khói bụi với các hạt bụi ảo tan biến sạch sẽ.
- Nút gạt chuyển chế độ lái sang Xanh kích hoạt làn sóng xanh ngọc lan tỏa.
- **Phân cảnh Dẫn đầu:** Đội hình xe xếp 2 hàng ngang ngắn gọn phía trên, chiếc VF 8 màu trắng độc lập tăng tốc bứt phá ở làn dưới, lao vút về phía trước.
![12 Chorus VF8](assets/12_chorus_vf8.jpg)
![13 Không khói bụi](assets/13_chorus_khong_khoi_bui.jpg)
![14 Chế độ xanh](assets/14_chorus_che_do_xanh.jpg)
![15 Dẫn đầu](assets/15_chorus_dan_dau.jpg)

#### Phân đoạn 8: Outro - Go Green & STOP (27.9s – 29.8s)
- Lời hát: *“Go Green! Chạm tương lai! STOP!”*
- Video VF 9 lăn bánh giữa cánh đồng năng lượng gió xanh ngát.
- Hiệu ứng sóng nước phát tỏa 2 đợt từ tâm chữ "O" của "GO".
- Đóng dấu chữ "STOP" viền đỏ dứt khoát trên màn hình đóng băng, sau đó là 1 frame tối hoàn toàn theo nhịp trống câm của bài hát.
![16 Go Green Ripples](assets/16_outro_go_green_ripples.jpg)
![17 STOP Freeze](assets/17_outro_stop_freeze.jpg)

#### Phân đoạn 9: Afterburn & End Card Tổng kết (29.9s – 38.4s)
- 12 nhịp trống dồn dập sau bài hát lướt qua từng góc cạnh xe.
- **End Card hoàn chỉnh:**
  - Khối Logo V chrome 3D và chữ VINFAST căn đối xứng quang học ở trung tâm.
  - Hai vệt quét sáng glint lướt qua cánh chim V.
  - Slogan *"GO GREEN · CHẠM TƯƠNG LAI"* với 2 vạch nhấn màu Xanh & Đỏ thương hiệu.
  - Đội hình cả 6 mẫu xe (VF 3 đến VF 9) dàn hàng ngang chuẩn theo kích thước thật, phản chiếu bóng gương trên sàn showroom sang trọng.
![18 End Card](assets/18_outro_endcard_lineup.jpg)

---

## 3. THIẾT LẬP MÀU SẮC & BRAND IDENTITY CHUẨN VINFAST

| Thành phần | Mã màu HEX / CSS | Ý nghĩa & Ứng dụng |
|---|---|---|
| **VinFast Brand Blue** | `#1464F4` | Màu chủ đạo, thanh chém vát, nền phát sáng |
| **Accent Energy Red** | `#EE3D43` | Điểm nhấn tốc độ: mép chuyển cảnh V-chevron, chữ BỨT PHÁ, STOP |
| **Eco Emerald Green**| `#2DBE78` / `#7CF0B2` | Dành riêng cho phân đoạn "Chế độ xanh", "Go Green" |
| **Background Ink** | `#05070D` / `#0B1330` | Màu tối trung tính điện ảnh thay cho màu xanh tím thông thường |
| **Chữ & Typography** | Font **Mulish** (500, 700, 800, 900) | Font chuẩn nhận diện thương hiệu VinFast toàn cầu |

---

## 4. BỘ FILES ĐÍNH KÈM & ĐƯỜNG DẪN LƯU TRỮ

Thư mục đã được đóng gói đầy đủ trên máy cục bộ tại:
- **Đường dẫn thư mục quy trình:**  
  `file:///Users/mac/Desktop/untitled%20folder%202/quy_trinh_vinfast_mv/`
- **Tài liệu Markdown hoàn chỉnh:**  
  `file:///Users/mac/Desktop/untitled%20folder%202/quy_trinh_vinfast_mv/QUY_TRINH_SAN_XUAT_VINFAST_MV.md`
- **Ảnh tổng quan 12 khung hình (Contact Sheet):**  
  `file:///Users/mac/Desktop/untitled%20folder%202/quy_trinh_vinfast_mv/assets/tong_quan_quy_trinh_contact_sheet.jpg`
- **Thư mục 18 ảnh chụp minh họa từng bước:**  
  `file:///Users/mac/Desktop/untitled%20folder%202/quy_trinh_vinfast_mv/assets/`
- **File video thành phẩm MP4 chất lượng cao:**  
  `file:///Users/mac/Desktop/untitled%20folder%202/VinFast_Cham_Tuong_Lai_v8_1080p60.mp4`

---

---

## 5. GIAO LƯU KẾT NỐI & CỘNG ĐỒNG CREATOR

> 🌟 **Lời chúc & Lời ngỏ kết nối:**  
> Kính chúc Quý Anh/Chị và toàn thể anh em Creator luôn dồi dào năng lượng sáng tạo, không ngừng bứt phá và cùng nhau kiến tạo nên những thước phim Kinetic Motion, TVC kỹ xảo đồ họa đỉnh cao, chạm tới những chuẩn mực quốc tế!  
>  
> Thân mời Anh/Chị tham gia nhóm Zalo kết nối, giao lưu học hỏi và chia sẻ tài nguyên video kỹ thuật cao:  
> 🔗 **Nhóm Zalo giao lưu & hỗ trợ:** [https://zalo.me/g/rnxe2yq8xrh2dawjbnrq](https://zalo.me/g/rnxe2yq8xrh2dawjbnrq)

---
*Tài liệu được khởi tạo tự động bởi Antigravity Motion Engine.*
