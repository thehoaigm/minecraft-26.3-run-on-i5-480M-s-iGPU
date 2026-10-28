# Kết quả kiểm thử rendering và diagnostics

Ngày khảo sát: 2026-10-28. Các phép đo dưới đây là diagnostics giới hạn của môi trường, không phải benchmark Minecraft hoặc benchmark prototype C++.

## Kiểm tra đã chạy

| Kiểm tra | Kết quả thực tế | Ý nghĩa và giới hạn |
|---|---|---|
| Inventory Minecraft client | Tệp client 26.3 có 41,483,720 bytes; SHA-1 `e877b6a07acd633fb3bb475002175cec036e7b87` khớp metadata cục bộ | Xác minh artifact đúng version đang được instance tham chiếu; không phải kiểm tra gameplay |
| Prism asset index 34 | 5,147 object paths được index; tất cả object được tham chiếu hiện có, 0 byte thiếu theo expected size | Kiểm tra các assets của version index này; không đánh giá asset nội dung hiển thị |
| CPU/OS/RAM | Windows 10 Home x64 build 19045; Intel Core i5 M 460, 2 core/4 logical, x86-64 SSE2/SSE3/SSE4.1, không AVX; 4,083,867,648 bytes RAM vật lý | Kết quả probe hệ thống; free RAM tại thời điểm probe khoảng 691 MB, dao động theo ứng dụng |
| WGL/OpenGL context | Context ẩn tạo được Microsoft Corporation / GDI Generic, GL 1.1.0, GLSL string null; không có WGL_ARB_create_context | Không phải Intel ICD và không xác minh maximum GL của Intel HD. Không truy vấn feature limits từ enum hiện đại trong context 1.1 |
| Vulkan loader | `LoadLibraryA("vulkan-1.dll")` thất bại với Win32 error 126; `vulkaninfo` không có; không thấy ICD registry/env trong phạm vi kiểm tra | Loader/ICD Vulkan chưa usable từ probe hiện tại. Chưa enumerate Vulkan device; không kết luận GPU không có khả năng Vulkan |
| D3D12 WARP | `EnumWarpAdapter` thành công; adapter Microsoft Basic Render Driver; `D3D12CreateDevice` thành công, tối đa feature level 12.1 đã truy vấn | Xác minh software D3D12 device trên OS. Không phải hardware acceleration và không chứng minh Mesa đã dùng WARP |
| Game launch/render | Chưa chạy | Không có menu/world/shader/runtime log. Không có FPS, frame-time, CPU/RAM gameplay hay correctness result |
| C++ prototype | Chưa được tạo hoặc build | Không có build/test pass; blocker được ghi trong `CPP_REIMPLEMENTATION_PLAN.md` |

## Kết quả benchmark

**Chưa có benchmark** Java hoặc C++. Không có FPS median, p50/p95/p99 frame-time, GPU/CPU usage trong game, draw call count hoặc chunk rebuild timing. Không diễn giải desktop resolution/driver diagnostic thành kết quả render.

## Thử nghiệm Java wrapper

Chưa chạy PoC. Không có Mesa Windows runtime hoặc SwiftShader binary trong môi trường; Vulkan loader không load được; chưa có quyền ghi để tạo Prism test instance bên ngoài workspace. Instance gốc không bị thay đổi. Mesa D3D12 -> D3D12 WARP mới ở mức candidate dựa trên tài liệu và probe WARP riêng.

## Tiêu chí kiểm thử tiếp theo

Chỉ ghi “đã chạy gameplay” khi log chứng minh context/backend và shader compile, vào world, chuyển động camera, tải chunk, entity/particle/transparency cơ bản không corruption đáng kể. Đo benchmark ở 854x480 với scene/cấu hình cố định, warm-up rồi chạy tối thiểu 60 giây ổn định. Java wrapper và C++ prototype cần kết quả riêng; menu-only không đủ.
