# 🚀 HƯỚNG DẪN DEPLOY NHANH — Streamlit Community Cloud

> **Mục tiêu:** Deploy app live trong **3–5 phút**, có URL public để nộp bài.
> **Nền tảng đề xuất:** **Streamlit Community Cloud** (`share.streamlit.io`)

---

## 🎯 Vì sao chọn Streamlit Community Cloud?

| Tiêu chí | Streamlit Cloud | Hugging Face Spaces | Render.com | Railway |
|---|:---:|:---:|:---:|:---:|
| Miễn phí | ✅ | ✅ | ⚠️ Giới hạn | ❌ Trả phí |
| Native Streamlit (không config) | ✅ | ⚠️ | ❌ | ❌ |
| Thời gian deploy | **2–3 phút** | 5–10 phút | 10–15 phút | 5 phút |
| Hỗ trợ `secrets.toml` qua UI | ✅ | ⚠️ | ⚠️ | ✅ |
| Auto-redeploy khi push | ✅ | ✅ | ✅ | ✅ |
| **Phù hợp deadline gấp** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |

→ **Streamlit Cloud** là lựa chọn tối ưu nhất: ít lỗi, ít config, deploy nhanh nhất.

---

## ✅ Checklist các file đã chuẩn bị sẵn (đã commit)

```
├── app.py                      ← Entry point (st.navigation)
├── requirements.txt            ← Đã pin version chuẩn
├── runtime.txt                 ← python-3.11 (tương thích tốt nhất)
├── packages.txt                ← libgl1, libglib2.0-0 (cho opencv)
├── .streamlit/
│   ├── config.toml             ← Theme + maxUploadSize
│   └── secrets.toml.example    ← Template cho GEMINI_API_KEY
└── pages_app/                  ← Các trang con
```

---

## 📋 CÁC BƯỚC DEPLOY (Làm theo đúng thứ tự)

### **Bước 1 — Lấy GEMINI API KEY (1 phút)**

1. Truy cập: <https://aistudio.google.com/app/apikey>
2. Đăng nhập Google → bấm **Create API key** → **Create API key in new project**
3. **Copy key** (dạng `AIzaSy...`) — lát nữa dán vào Streamlit Cloud

> 💡 Miễn phí, không cần credit card. Quota free đủ demo thoải mái.

---

### **Bước 2 — Đảm bảo branch đã được push lên GitHub (30 giây)**

Branch `claude/deploy-fraud-detection-NGpn3` đã có sẵn các file deploy. Kiểm tra:

```bash
git status               # phải sạch
git log --oneline -3     # xem commit deploy
git push origin claude/deploy-fraud-detection-NGpn3
```

> ⚠️ **Khuyến nghị:** Merge branch này vào `main` để Streamlit Cloud lấy code từ branch chính:
> ```bash
> git checkout main
> git merge claude/deploy-fraud-detection-NGpn3
> git push origin main
> ```

---

### **Bước 3 — Deploy lên Streamlit Cloud (2 phút)**

1. Truy cập: <https://share.streamlit.io>
2. Bấm **Sign in with GitHub** → cấp quyền cho Streamlit
3. Bấm nút **Create app** (góc trên bên phải) → chọn **Deploy a public app from GitHub**
4. Điền form:

   | Trường | Giá trị |
   |---|---|
   | **Repository** | `truchuynh10121996-gif/mlp-trong-bai-toan-ato` |
   | **Branch** | `main` (hoặc `claude/deploy-fraud-detection-NGpn3` nếu chưa merge) |
   | **Main file path** | `app.py` |
   | **App URL** *(tuỳ chọn)* | ví dụ: `siamese-fraud-demo` → URL sẽ là `https://siamese-fraud-demo.streamlit.app` |

5. **TRƯỚC KHI BẤM DEPLOY** → bấm **Advanced settings...**:
   - **Python version:** `3.11`
   - **Secrets:** dán nội dung sau (thay `AIza...` bằng key của bạn):
     ```toml
     GEMINI_API_KEY = "AIzaSy..............."
     # Tuỳ chọn — đổi model nếu cần:
     # GEMINI_MODEL = "gemini-2.5-flash"
     ```
6. Bấm **Save** → bấm **Deploy!**

---

### **Bước 4 — Chờ build (3–5 phút lần đầu)**

- Streamlit Cloud sẽ:
  1. Clone repo
  2. Cài system packages từ `packages.txt`
  3. Cài Python packages từ `requirements.txt` (lâu nhất — tensorflow-cpu ~500MB)
  4. Chạy `streamlit run app.py`
- Bạn sẽ thấy log streaming. Nếu thấy dòng `You can now view your Streamlit app...` → **THÀNH CÔNG** 🎉

---

## 🐛 XỬ LÝ LỖI THƯỜNG GẶP

### Lỗi: `ModuleNotFoundError: No module named 'cv2'` khi vào tab rPPG
→ Đảm bảo `packages.txt` đã có `libgl1`. Reboot app: **Manage app → Reboot**.

### Lỗi: `Killed` hoặc `Out of memory` khi train Siamese
→ TensorFlow nặng. Cách khắc phục nhanh:
- Reboot app sau khi train xong (Manage app → Reboot)
- Hoặc giảm `epochs` trong UI training Siamese

### Lỗi: `GEMINI_API_KEY not found`
→ Vào **Manage app → Settings → Secrets** → kiểm tra đã dán đúng định dạng TOML:
```toml
GEMINI_API_KEY = "AIzaSy..."
```
(có dấu ngoặc kép)

### Lỗi: `429 Quota exceeded` từ Gemini
→ Đổi model trong secrets:
```toml
GEMINI_MODEL = "gemini-2.5-flash-lite"
```

### App "sleeping" sau nhiều giờ không dùng
→ Bình thường với free tier. Truy cập lại URL → app tự wake up sau ~30 giây.

---

## 🎓 KỊCH BẢN DEMO KHI THUYẾT TRÌNH (gợi ý)

1. **Trang chủ** → giới thiệu tổng quan kiến trúc 3 tầng
2. **Kiến trúc Siamese Network — MLP** → train model live → show AUC/accuracy
3. **rPPG chống DeepFake** → upload video → show heart-rate signal
4. **Demo End-to-End** → flow hoàn chỉnh
5. **AI hỗ trợ** → chat với Gemini về kết quả

---

## 🔄 CÁCH UPDATE CODE SAU KHI ĐÃ DEPLOY

Streamlit Cloud auto-redeploy khi bạn push code lên GitHub:

```bash
git add .
git commit -m "update demo"
git push origin main          # hoặc branch đã chọn lúc deploy
```

→ App tự rebuild trong 1–2 phút.

---

## 🆘 PLAN B — Nếu Streamlit Cloud lỗi không khắc phục được

Backup nhanh sang **Hugging Face Spaces** (5 phút):

1. Tạo Space tại <https://huggingface.co/new-space>
2. Chọn SDK: **Streamlit**, Python: **3.11**
3. Clone Space repo, copy toàn bộ file dự án vào, push
4. Thêm secret `GEMINI_API_KEY` ở tab **Settings → Variables and secrets**

---

**Chúc bạn deploy thành công, nộp bài đúng hạn!** 🍀
