# N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
**Môn học:** Phát triển ứng dụng với mã nguồn mở-TEE0421   
**Sinh viên thực hiện:** Nguyễn Thế Dương     
**Mã số sinh viên:** K225480106007  
**Lớp:** K58KTP
# AUTO_N8N_WORDPRESS
## 1. TỔNG QUAN HỆ THỐNG (ARCHITECTURE OVERVIEW)
Hệ thống xây dựng một luồng công việc tự động hóa hoàn chỉnh (Workflow Automation) từ khâu tiếp nhận yêu cầu, xử lý ngôn ngữ tự nhiên bằng Trí tuệ nhân tạo (AI), chuẩn hóa cấu trúc dữ liệu và đồng bộ xuất bản lên nền tảng Web CMS trong thời gian thực.

**Các thành phần cốt lõi:**
1. **Webhook / Trigger Input:** Telegram Bot API (Đóng vai trò tiếp nhận Interface từ người dùng).
2. **AI Generation Engine:** Google Gemini Pro Model via API (Xử lý prompt, sinh bài viết chuẩn SEO chuyên sâu).
3. **Data Transformation Layer:** Code Node JavaScript trong n8n (Phân tách cấu trúc dữ liệu thô, lọc Header, tách tiêu đề và nội dung HTML).
4. **Content Management System (CMS):** WordPress REST API chạy trên môi trường Docker Isolation (Nơi lưu trữ và hiển thị bài viết).
## Các bước thực hiện
### 1. Cấu hình file **docker-compose.yml**

<img width="1472" height="758" alt="image" src="https://github.com/user-attachments/assets/0a621ec6-8304-4665-bfff-0f4d12974d51" />
Cách khởi chạy:
Mở terminal trên Ubuntu, điều hướng vào thư mục chứa file docker-compose.yml và chạy lệnh:**docker compose up -d**

- phpMyAdmin sẽ truy cập được tại: http://localhost:8081. Bạn có thể đăng nhập bằng user(****) và pass(********) để quản lý cơ sở dữ liệu (Database wordpress_db đã được tạo sẵn qua biến môi trường).

<img width="1909" height="920" alt="image" src="https://github.com/user-attachments/assets/df8de834-0437-49ea-b51a-c10f2b63ed0c" />

- WordPress sẽ chạy tại: http://localhost:8080. Bạn truy cập vào đây để thực hiện các bước cài đặt WordPress ban đầu (chọn ngôn ngữ, tạo tài khoản admin).
- n8n lúc này sẽ hoạt động nội bộ tại cổng: **http://localhost:5678.**
  
### 2. Cấu hình Route trên Cloudflare Tunnel cho n8n
 Public n8n qua Cloudflare Tunnel

 1. Truy cập Cloudflare Zero Trust
- Vào **Networks → Tunnels**
- Chọn Tunnel đã tạo trước đó
- Bấm **Edit**
<img width="1892" height="909" alt="image" src="https://github.com/user-attachments/assets/774084c0-08ec-479b-a737-d9eec5648ce3" />

---

 2. Thêm Public Hostname
- Chuyển sang tab **Public Hostname**
- Bấm **Add a public hostname**

---

 3. Cấu hình Hostname mới
- **Subdomain**: `n8n`  
- **Domain**: `wapvip.io.vn`  
- **Service**:  
  - Type: `HTTP`  
  - URL: `localhost:5678` (cổng của n8n)  
<img width="1894" height="858" alt="Untitled8" src="https://github.com/user-attachments/assets/fa75c831-723f-4aba-91aa-2326c0d2ec99" />

---
 4. Lưu cấu hình
- Bấm **Save hostname**  
- Truy cập giao diện n8n qua:  
  `https://n8n.wapvip.io.vn`

---

 5. Hoàn tất
- Tạo tài khoản **admin n8n** lần đầu trên giao diện web.  
- Từ đây, n8n có thể nhận dữ liệu kích hoạt (**Webhooks**) từ internet ổn định qua subdomain mới.






### 3. Cấu hình xác thực API trên WordPress (Application Password)

 Tạo mật khẩu ứng dụng cho WordPress

 1. Đăng nhập trang quản trị
- Truy cập: `https://blog.wapvip.io.vn/wp-admin`

---

 2. Vào mục hồ sơ người dùng
- Chọn **Users (Thành viên) → Profile (Hồ sơ của bạn)**

---

 3. Tạo mật khẩu ứng dụng
- Cuộn xuống cuối trang, tìm phần **Application Passwords (Mật khẩu ứng dụng)**  
- Nhập tên gợi nhớ (ví dụ: `n8n-token`) vào ô **New Application Password Name**  
- Bấm nút **Add New Application Password**

---

 4. Nhận mật khẩu
- WordPress sẽ sinh ra một chuỗi mật khẩu ngẫu nhiên dạng:  
  `xxxx xxxx xxxx xxxx xxxx`  
- **Lưu ý**: mật khẩu này chỉ hiển thị **một lần duy nhất**  
- Hãy copy và lưu lại để sử dụng cho ứng dụng bên ngoài (ví dụ: n8n)

---

 ✅ Kết quả
- Ứng dụng bên ngoài như **n8n** có thể dùng mật khẩu này để đăng bài hoặc thao tác với WordPress một cách bảo mật.

<img width="1896" height="988" alt="Untitled9" src="https://github.com/user-attachments/assets/eed58180-8f76-49b0-a6ae-798c7dfe3720" />

### Bước 4: Thiết lập Workflow đăng bài tự động trên n8n

Một Workflow cơ bản thường gồm 2 **Nodes** chính:
- **Nguồn dữ liệu vào** (Input Node)
- **WordPress Output Node** (đăng bài)

---

 1. Tạo liên kết thông tin (Credentials)
- Set up onwn account  
<img width="1914" height="966" alt="Untitled" src="https://github.com/user-attachments/assets/603ec40c-2d3d-4498-b5ed-60d59655fba0" />
- Giao diện của **N8N**
<img width="1898" height="951" alt="Untitled1" src="https://github.com/user-attachments/assets/e5361a3b-0b5c-4716-9d17-28b3bf10aa02" />

- Trên giao diện n8n, chọn **Credentials** từ menu trái → **Add Credential**
- Tìm chọn **WordPress REST API**

2. Cấu hình chi tiết:
- **URL**: `https://blog.wapvip.io.vn/wp-json/wp/v2`
- **Authentication**: chọn `Basic Auth`
- **User**: nhập tên đăng nhập **Admin WordPress**
- **Password**: dán chuỗi **Application Password** (24 ký tự) đã tạo ở Bước 3  
  *(Lưu ý: không dùng mật khẩu đăng nhập tài khoản thông thường)*  
<img width="1916" height="1022" alt="image" src="https://github.com/user-attachments/assets/a15a4c27-4f93-4eb0-9349-6889b4ac2366" />

- Bấm **Save** để lưu Credential.

---
## ✅ Kết quả
- Bạn đã tạo thành công Credential để n8n có thể kết nối với WordPress qua REST API.  
- Đây là bước nền tảng để Workflow có thể đăng bài tự động từ dữ liệu đầu vào.

### Mật khẩu ứng dụng WordPress
<img width="1918" height="954" alt="Untitled5" src="https://github.com/user-attachments/assets/a3bb907d-1217-439c-9843-8bd450e5b6a0" />

- Để rolo với quyền cao nhất Administrator
<img width="1885" height="688" alt="Untitled11" src="https://github.com/user-attachments/assets/bb66c62b-67e9-474a-9cbf-6e508d3f26a3" />

### Token Bot Telegram
<img width="1903" height="955" alt="Untitled4" src="https://github.com/user-attachments/assets/94e4c1b6-a79b-4174-9903-ccaf20571b90" />

- Lấy **@botfather** + key token 
  <img width="678" height="688" alt="Untitled12" src="https://github.com/user-attachments/assets/0a4e441f-6490-49be-8850-3451ecb418bb" />


### Google Gemini API Key:
<img width="1900" height="956" alt="Untitled6" src="https://github.com/user-attachments/assets/895bbffe-446b-4d00-9c17-7d74dc11139a" />

#### Lưu ý:
- Nên dùng model: **gemini-3.5-flash** (khôn hơn mấy model khác :))
  <img width="1854" height="834" alt="Untitled10" src="https://github.com/user-attachments/assets/afeba05b-0845-4067-b339-2d6cefe03e1a" />










## 2. QUY TRÌNH THIẾT LẬP VÀ CẤU HÌNH CHI TIẾT (STEP-BY-STEP IMPLEMENTATION)
### BƯỚC 1: Xây Dựng Hạ Tầng Docker Môi Trường Cục Bộ
Thiết lập và khởi chạy các container phục vụ cho hệ thống bao gồm n8n và WordPress thông qua file cấu hình `docker-compose.yml`.

* **Xử lý lỗi hệ thống ngầm (Xung đột Webhook và REST API):**
    * Do Telegram ép buộc kết nối bảo mật lớp mã hóa, biến môi trường của n8n được tinh chỉnh cấu hình định danh sang tên miền Public thay vì dùng IP Local để định tuyến Webhook:
        ```yaml
        - N8N_HOST=n8n.wapvip.io.vn
        - WEBHOOK_URL=[https://n8n.wapvip.io.vn/](https://n8n.wapvip.io.vn/)
        ```
    * Sửa lỗi máy chủ Web Apache trong Docker tự động nuốt chuỗi mã xác thực nâng cao (`Authorization Header`) bằng cách cấu hình bổ sung quy tắc ghi đè định tuyến trong tệp cấu hình `.htaccess` của WordPress:
        ```text
        RewriteEngine On
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
        ```

### BƯỚC 2: Cấu Hình Telegram Trigger Node (Interface Input)
1. Khởi tạo Telegram Bot qua `@BotFather` để lấy chuỗi bảo mật **Access Token**.
2. Trong n8n, thiết lập thông số cho nút Telegram Trigger:
    * **Credential for Telegram API:** Chọn Token tương ứng đã cấu hình.
    * **Trigger On:** Thiết lập chính xác về sự kiện `message` để hệ thống chỉ kích hoạt khi phát hiện có chuỗi văn bản lệnh mới được gửi tới, tránh lặp luồng vô hạn bởi các event tương tác phụ (emoji, edit).
3. *Giải pháp kiểm thử cục bộ:* Sử dụng tính năng `Set Mock Data` để giả lập gói tin JSON mẫu của Telegram gửi về nhằm cấu hình tối ưu cấu trúc dữ liệu mà không bị nghẽn mạng API.
    ```json
    {
      "message": {
        "text": "Viết một bài blog ngắn phân tích lợi ích của Docker Compose trong phát triển phần mềm"
      }
    }
    ```

### BƯỚC 3: Cấu Hình AI Model Node (Google Gemini)
1. Khởi tạo nút **`Message a model`** kết nối với hạ tầng API Google AI Studio.
2. **Tối ưu hóa lựa chọn Model:** Thay thế dòng `gemini-3-flash-preview` (tốc độ cao nhưng xử lý ngôn ngữ nông, dễ lan man) sang dòng model cao cấp **`gemini-1.5-pro`** hoặc **`gemini-2.0-pro`** để tăng cường khả năng tư duy logic và chiều sâu kiến thức công nghệ.
3. **Chuyển đổi kiểu dữ liệu Prompt (Mấu chốt logic):** Ô dữ liệu Prompt chuyển từ chế độ chuỗi thô (`Fixed Text`) sang biểu thức lập trình hệ thống (`Expression Mode` - Kích hoạt bằng cú pháp `fx`). 
4. Áp dụng kỹ thuật nối chuỗi (String Concatenation) để đóng gói System Prompt quy định vai trò chuyên gia SEO kết hợp linh hoạt với biến động tin nhắn đầu vào từ Telegram:
    ```text
    {{ "Hãy đóng vai là một chuyên gia Cloud Engineer và Blogger chuẩn SEO chuyên nghiệp. Hãy viết một bài viết công nghệ chi tiết, đi thẳng vào trọng tâm, phân tích sâu, không chào hỏi rườm rà. Định dạng hoàn toàn bằng các thẻ HTML (<h2>, <p>, <strong>). Chủ đề bài viết lấy từ Telegram: " + $json.message.text }}
    ```


### BƯỚC 4: Chuẩn Hóa Cấu Trúc Dữ Liệu Qua Code Node JavaScript
Khi nhận kết quả văn bản thô đổ về từ cấu trúc cây thư mục JSON của Gemini (`json.content.parts[0].text`), dữ liệu cần được bóc tách tự động để chuyển giao chuẩn hóa sang các tham số đầu vào mà WordPress yêu cầu.
Đoạn mã xử lý an toàn tuyệt đối (`Runtime Error Protection`) được triển khai tại nút **`Code in JavaScript`**:
```javascript
let aiResponse = "";
try {
    if ($node["Message a model"].json.content && $node["Message a model"].json.content.parts) {
        aiResponse = $node["Message a model"].json.content.parts[0].text;
    } else {
        aiResponse = $node["Message a model"].json.text || $node["Message a model"].json.output || "";
    }
} catch (e) {
    aiResponse = "Không thể đọc dữ liệu từ Gemini.";
}

let title = "Bài viết tự động từ AI";
let content = aiResponse;

if (aiResponse && aiResponse.trim() !== "") {
    // Thuật toán Regex quét và bóc tách thẻ heading đầu tiên làm tiêu đề bài viết
    const titleMatch = aiResponse.match(/<h[1-2]>(.*?)<\/h[1-2]>/);
    
    if (titleMatch && titleMatch[1]) {
        title = titleMatch[1].replace(/<\/?[^>]+(>|$)/g, "").trim(); 
        content = aiResponse.replace(titleMatch[0], ""); // Loại bỏ tiêu đề khỏi nội dung để tránh trùng lặp
    } else {
        const lines = aiResponse.split('\n');
        title = lines[0].replace(/<\/?[^>]+(>|$)/g, "").trim();
        if (title.length > 100) {
            title = title.substring(0, 95) + "...";
        }
    }
}

if (!title || title.trim() === "") title = "Bài viết về Công nghệ mới";

return {
    json: {
        title: title,
        content: content
    }   
};
 ```
  /<br>
  
### BƯỚC 5 : KẾT QUẢ
#### 1. Workflow
<img width="1902" height="911" alt="image" src="https://github.com/user-attachments/assets/813eb560-a23a-4098-a3a9-245886c9e53d" />

#### 2. Chat với bot telegram
<img width="1911" height="1079" alt="image" src="https://github.com/user-attachments/assets/57f548f4-fdc8-40bf-98dd-6139468225b2" />

#### 3. Các post đã được gemini viết
<img width="1904" height="1022" alt="image" src="https://github.com/user-attachments/assets/c60cac00-88a7-4462-aa9e-5baf18905353" />

#### Một số bài demo do Al đã viết
1. Link :[https://blog.wapvip.io.vn/wp-admin/post.php?post=45&action=edit](https://blog.wapvip.io.vn/diem-giao-thoa-giua-vien-tuong-va-thuc-te-tai-sao-dan-it-can-xem-phim-sci-fi/)
<img width="1902" height="1025" alt="image" src="https://github.com/user-attachments/assets/5b5c36e0-9607-441e-98aa-fe24fc933d2f" />
2. link: https://blog.wapvip.io.vn/ap-luc-kep-tu-giang-duong-den-doanh-nghiep-bai-toan-can-bang-cua-sinh-vien-it/
<img width="1914" height="1021" alt="image" src="https://github.com/user-attachments/assets/58c0181c-4171-4c34-abae-45d0aea8bbc1" />

3. Link: https://blog.wapvip.io.vn/de-tang-cho-ban-gai-ten-ngoc-anh-ban-co-the-chon-mot-trong-nhung-bai-tho-duoi-day-tuy-thuo/
<img width="1912" height="1027" alt="image" src="https://github.com/user-attachments/assets/6ab31a67-48a0-4c6c-a3e5-2106e46c68cb" />














