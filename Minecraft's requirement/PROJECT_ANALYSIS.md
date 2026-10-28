# Phân tích workspace Minecraft Java 26.3 trên máy có giới hạn OpenGL

Ngày khảo sát: 2026-10-28  
Workspace: `C:\Users\theho\AppData\Roaming\PrismLauncher\instances\26.3`

## Tóm tắt

Workspace là PrismLauncher OneSix instance cho Minecraft Java 26.3 và LWJGL 3.4.3, không phải repository source. Client JAR và các runtime/assets đúng version được tham chiếu ở thư mục shared của PrismLauncher. Không có loader mod, mod JAR, resource pack, shader pack, game log hoặc source Minecraft đầy đủ.

Máy đã xác minh là Windows 10 Home x64 build 19045, Intel Core i5 M 460 (2 core/4 thread), RAM 4 GiB và Intel HD Graphics với driver 2012. WGL probe chỉ tạo Microsoft GDI Generic OpenGL 1.1; đây không phải Intel ICD, nên native Intel GL capability vẫn chưa xác minh. Vulkan loader không load được trong probe. D3D12 WARP software adapter đã tạo được với feature level tối đa 12.1.

JAR Minecraft 26.3 có cả OpenGL và Vulkan backend; release notes xác nhận SDL3, ShaderC cho OpenGL/Vulkan, GLSL changes và MultiDrawIndirect trên thiết bị hỗ trợ. Shader terrain thực đóng gói dùng GLSL 330. Do đó native GL 2.1/GLSL 1.20 không thể được xem là đủ cho shader path hiện có. Tắt OIT có thể tránh OIT path tùy chọn, nhưng không thay đổi terrain shader version.

Ứng viên Java có chuỗi API hợp lý nhất là **Minecraft OpenGL → Mesa Gallium D3D12 → D3D12 WARP**: Mesa D3D12 công bố desktop OpenGL 3.3, còn WARP đã được kiểm tra trên máy. Đây chưa phải PoC: chưa có Mesa runtime, chưa tích hợp SDL3/LWJGL, chưa compile shader trong game và chưa chạy gameplay. Hiệu năng CPU chưa đo.

Không có source/license Minecraft để port. Hướng C++ hợp lệ là clean-room voxel sandbox, không phải bản Minecraft đầy đủ và không tương thích Java mods trực tiếp. Project C++ không được tạo vì đường Documents nằm ngoài writable root của phiên làm việc; không đặt source vào instance để né giới hạn.

## Trạng thái triển khai và kiểm thử

- Instance gốc không bị thay đổi về launcher setting, JAR, mods, DLL hệ thống hoặc driver; các thay đổi trong workspace chỉ là tài liệu khảo sát được yêu cầu.
- Không launch Minecraft, không tạo test instance, không chạy game/Java wrapper PoC.
- Không có C++ project/build/prototype. Không có CMake, Meson, Ninja hoặc Mesa Windows runtime được phát hiện; `g++ 14.1` có sẵn nhưng không đủ tự nó để build prototype.
- Đã chạy diagnostics riêng cho WGL, Vulkan loader, CPU/RAM/OS và D3D12 WARP. Chưa có benchmark FPS/frame-time hoặc kết quả gameplay.
- Git: `not a git repository`.

## Báo cáo chi tiết

- [MINECRAFT_WORKSPACE_AUDIT.md](./MINECRAFT_WORKSPACE_AUDIT.md) — inventory, metadata, runtime, hardware/API và phần chưa xác minh.
- [RENDERING_REQUIREMENTS.md](./RENDERING_REQUIREMENTS.md) — pipeline trong JAR/notes, GL feature matrix và mismatch GL 2.1.
- [JAVA_WRAPPER_PLAN.md](./JAVA_WRAPPER_PLAN.md) — API chains, candidate ranking, PoC gate, rollback và blocker.
- [MOD_COMPATIBILITY_MATRIX.md](./MOD_COMPATIBILITY_MATRIX.md) — phạm vi mod thực tế (không có mod JAR).
- [CPP_REIMPLEMENTATION_PLAN.md](./CPP_REIMPLEMENTATION_PLAN.md) — clean-room scope, kiến trúc và lý do project chưa thể tạo.
- [RENDER_TEST_RESULTS.md](./RENDER_TEST_RESULTS.md) — diagnostics đo được và các bài test chưa chạy.

## Bằng chứng và nguồn kỹ thuật chính

- Minecraft 26.3 release notes: https://www.minecraft.net/en-us/article/minecraft-java-edition-26-3
- Minecraft Java system requirements: https://www.minecraft.net/en-us/article/minecraft-java-edition-system-requirements
- Mesa D3D12: https://docs.mesa3d.org/drivers/d3d12.html
- Microsoft WARP adapter: https://learn.microsoft.com/en-us/windows/win32/api/dxgi1_4/nf-dxgi1_4-idxgifactory4-enumwarpadapter
- Khronos OpenGL registry: https://registry.khronos.org/OpenGL/index_gl.php

Các báo cáo phân biệt bằng chứng workspace, phép đo, suy luận và điểm chưa xác minh. Không có số FPS giả định; không kết luận Minecraft chạy được chỉ từ mở menu hoặc từ việc tạo thành công WARP device.
