---
name: code-explainer-vi
description: Giải thích code bằng tiếng Việt một cách rõ ràng và có cấu trúc. Phân tích chức năng, luồng logic, pattern sử dụng, edge case và đưa ra ví dụ. Use when users ask to explain code in Vietnamese, giải thích code, phân tích function, code này làm gì, đọc hiểu code, hoặc mention giải thích bằng tiếng Việt.
---

# Code Explainer (Vietnamese)

Skill này giúp giải thích code bằng tiếng Việt một cách có hệ thống, phù hợp cho việc học code, đọc codebase lạ, hoặc onboard dự án mới.

## Instructions

Khi được kích hoạt, hãy giải thích đoạn code theo **5 phần** sau, luôn dùng tiếng Việt:

### 1. 📋 Tóm tắt ngắn gọn (1-2 câu)
Code này làm gì ở mức tổng quát? Viết cô đọng, không dùng thuật ngữ khó.

### 2. 🔍 Phân tích chi tiết
Đi qua code theo trình tự, giải thích:
- **Input**: tham số nhận vào là gì, kiểu dữ liệu, ý nghĩa
- **Luồng xử lý**: từng bước code làm gì, tại sao
- **Output**: trả về gì, kiểu dữ liệu

Sử dụng bullet list hoặc đánh số bước cho dễ theo dõi.

### 3. 🎯 Pattern & Kỹ thuật sử dụng
Nếu code có áp dụng pattern/kỹ thuật đáng chú ý, hãy chỉ ra:
- Design pattern (Factory, Observer, Singleton, ...)
- Kỹ thuật (recursion, memoization, closure, async/await, ...)
- Thuật toán (binary search, DP, two-pointer, ...)

Nếu code đơn giản không có gì đặc biệt, **bỏ qua phần này**.

### 4. ⚠️ Edge case & Lưu ý
Chỉ ra những trường hợp đặc biệt code xử lý hoặc CHƯA xử lý:
- Input rỗng, null, undefined
- Giá trị âm, 0, số lớn
- Lỗi tiềm ẩn (race condition, off-by-one, ...)

### 5. 💡 Ví dụ minh họa
Cho 1-2 ví dụ input cụ thể và output tương ứng để người đọc hình dung.

## Nguyên tắc viết

- **Dùng tiếng Việt tự nhiên**, giữ nguyên thuật ngữ kỹ thuật tiếng Anh khi cần (function, array, loop, callback...)
- **Tránh dịch máy móc** (ví dụ: đừng dịch "array" thành "mảng" nếu nghe không tự nhiên)
- **Ngắn gọn, đi thẳng vào trọng tâm** — không giải thích lan man
- **Dùng code block** khi trích dẫn code
- **Dùng emoji tiêu đề** để chia section rõ ràng

## Ví dụ Output

Khi user đưa đoạn code JavaScript:
```js
const debounce = (fn, delay) => {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
};
```

### 1. 📋 Tóm tắt
Hàm `debounce` giúp hoãn việc gọi một function cho đến khi user ngừng kích hoạt trong một khoảng thời gian (`delay` ms).

### 2. 🔍 Phân tích chi tiết
- **Input**: `fn` (function cần debounce), `delay` (số ms chờ)
- **Luồng xử lý**:
  1. Tạo biến `timer` lưu id của setTimeout
  2. Trả về một function mới nhận bất kỳ tham số nào (`...args`)
  3. Mỗi lần function mới được gọi: hủy timer cũ (`clearTimeout`), tạo timer mới
  4. Sau `delay` ms không gọi lại, `fn` sẽ chạy với args mới nhất
- **Output**: function đã được debounce

### 3. 🎯 Pattern
- **Closure**: `timer` được giữ trong scope của function trả về
- **Higher-order function**: nhận function, trả về function

### 4. ⚠️ Edge case
- Nếu user gọi liên tục, `fn` sẽ không bao giờ chạy
- Không xử lý `this` binding (nếu cần dùng với method cần thêm `.apply(this, args)`)

### 5. 💡 Ví dụ
```js
const search = debounce((q) => console.log("search:", q), 300);
search("a"); search("ab"); search("abc");
// Sau 300ms chỉ in "search: abc"
```
