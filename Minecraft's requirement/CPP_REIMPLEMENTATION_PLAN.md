# Kế hoạch tái triển khai clean-room bằng C++

Ngày khảo sát: 2026-10-28  
Project đích được yêu cầu: `C:\Users\theho\Documents\mc26.3-cpp-rewrite`.

## Source và phạm vi pháp lý

**Đã xác minh:** Không có source tree hoặc license source Minecraft trong workspace. Client JAR 26.3 ở Prism shared libraries có bytecode/resources đã đóng gói; nó không phải repository source đầy đủ. Không giải nén/copy mã hay tài nguyên game sang project C++.

Vì không có source đầy đủ cùng license thích hợp, phương án được phép lập kế hoạch là một **clean-room voxel sandbox** độc lập, không tuyên bố là bản port/rewrite hoàn chỉnh của Minecraft. Dùng procedural colors/placeholder textures; mọi tài nguyên ngoài project chỉ được đọc khi người dùng cung cấp hợp pháp. Java mods, Fabric/Forge/NeoForge APIs, Mixin, event bus, reflection và JNI không chạy trực tiếp trong C++ process; cần bridge/JVM hoặc triển khai lại API/hành vi ở quy mô riêng.

## Blocker triển khai trong phiên này

**Bị chặn:** Hệ thống chỉ cấp quyền ghi trong instance workspace và thư mục tạm. Đường Documents ở trên nằm ngoài writable root. Không tạo project thay thế bên trong instance vì đặc tả cấm đặt source C++ ở runtime instance; không thử ghi vượt quyền. Đường đích đã được inventory và chưa tồn tại tại thời điểm kiểm tra, nên không có nội dung nào cần giữ lại. Để tạo source đúng vị trí, workspace cần được mở với quyền ghi rõ ràng vào thư mục Documents đó.

**Bị chặn kỹ thuật bổ sung:** `g++ 14.1` có sẵn, nhưng `cmake`, `ninja`, `meson` và `pkg-config` không có trong PATH. Không có Mesa DLL/build hiện diện. WGL diagnostic probe chỉ tạo được Microsoft GDI Generic GL 1.1, không chứng minh Intel driver hỗ trợ GL 2.1. Do đó chưa thể build hoặc chạy prototype GL 2.1 trên driver thực tế.

## Thiết kế vertical slice đề xuất

1. CMake executable Windows x64; khởi động chẩn đoán adapter, GL vendor/renderer/version/GLSL và extension cần thiết.
2. Tạo context tương thích GL 2.1; khởi động dừng với thông báo rõ khi context hoặc GLSL 1.20 không có. Tạo context qua SDL3 nếu có dependency và loader phù hợp; nếu SDL3 không sẵn có, chọn window layer chỉ sau khi kiểm tra trên OS/toolchain đích.
3. Shader GLSL 1.20, bind attribute locations trước link, giữ compile/link logs.
4. CPU-generated indexed chunk mesh; chỉ sinh mặt block có hàng xóm rỗng; gộp geometry thành VBO/IBO và draw-call theo chunk. Không dùng VAO/UBO/texture-array/compute/MDI nếu chưa detect extension.
5. Procedural block palette hoặc texture atlas placeholder, depth test, face culling, alpha blending đơn giản.
6. First-person camera, keyboard/mouse, giới hạn world nhỏ và render distance để kiểm soát RAM/CPU.
7. HUD/diagnostics: FPS, frame-time, chunk count, vertex/index count, draw call ước tính và thời gian rebuild mesh.

## Mục tiêu kiểm thử, chưa phải kết quả

Target thử nghiệm ban đầu: 854x480, scene voxel nhỏ, mục tiêu quan sát gameplay ổn định gần 30 FPS trên phần cứng mục tiêu. Chưa đo được FPS; không bảo đảm đạt mục tiêu. Sau khi có project và build được, đo frame-time p50/p95/p99, median FPS, CPU/RAM, draw calls và chunk mesh rebuild với cùng scene, warm-up và ít nhất 60 giây.

## Roadmap riêng cho C++

- **Slice 0:** diagnostic/context validation và cửa sổ.
- **Slice 1:** camera, mesh chunk tĩnh, placeholder materials, depth/culling.
- **Slice 2:** interaction, collision, chunk rebuild cục bộ và đo profiling.
- **Slice 3:** greedy meshing, frustum/distance culling, cache, texture atlas và chunk streaming.
- **Slice 4:** lighting, save/load, transparency sorting, entities/particles.
- **Xa hơn:** world generation/data compatibility có kiểm tra license; mod API parity là dự án lớn, không thuộc prototype.

## So sánh hai hướng

| Tiêu chí | Java wrapper | C++ clean-room sandbox |
|---|---|---|
| Mục tiêu | Giữ client Java 26.3, chuyển desktop OpenGL qua Mesa D3D12 đến D3D12 WARP | Renderer/gameplay độc lập target GL2.1/GLSL1.20 |
| Trạng thái | Đường chuyển đổi có cơ sở tài liệu và WARP device đã đo; chưa tích hợp client | Chưa tạo project hoặc prototype do sandbox không cho ghi vào Documents |
| Khả năng giữ gameplay Minecraft | Cao hơn nếu backend/shaders và launch thật sự qua; hiện chưa xác minh | Chỉ các chức năng tự triển khai; không phải Minecraft đầy đủ |
| Mod Java | Không có mod thực tế trong instance; từng mod cần thử | Không chạy trực tiếp; cần JVM/bridge hoặc reimplementation |
| Hiệu năng | Software WARP có thể nặng; chưa benchmark | Chưa build/benchmark; mục tiêu 30 FPS là target kiểm thử |
| Blocker chính | Chưa có Mesa runtime/build, context integration và game test; phần cứng yếu | Quyền ghi ngoài workspace, thiếu CMake/SDL3/driver GL2.1 đã xác minh và scope lớn |
| Kết luận | Ứng viên PoC ưu tiên nhất nếu chuẩn bị được runtime cách ly và đo hiệu năng | Clean-room route có thể kiểm soát API, nhưng chưa thể triển khai trong sandbox hiện tại |

## Các tệp project chưa tạo

Chưa tạo `README.md`, `ROADMAP.md`, `CMakeLists.txt`, `docs/GL21_DESIGN.md`, `docs/MOD_COMPATIBILITY.md` hoặc `src/main.cpp` vì chúng thuộc project ngoài writable root. Không tạo bản giả/placeholder trong instance.

## Nguồn thiết kế

- OpenGL registry/specifications (core versions, extensions, GLSL): https://registry.khronos.org/OpenGL/index_gl.php
- SDL3 GL context API: https://wiki.libsdl.org/SDL3/SDL_GL_CreateContext
- OpenGL 2.1/GLSL 1.20 là target thiết kế yêu cầu của prototype; capability thực tế phải đọc từ context của driver, không suy ra từ tên GPU.
