# Java wrapper / translation plan — Minecraft 26.3

Ngày khảo sát: 2026-10-28  
Instance gốc chưa bị sửa. Không có game launch hay wrapper PoC trong lần này.

## Đầu vào đã xác minh

- Instance metadata chỉ cài Minecraft 26.3 + LWJGL3.4.3; không có mod loader/mods.
- Client JAR có backend OpenGL và Vulkan cùng option preferredGraphicsBackend. Không có options.txt/game log; backend được chọn thực tế chưa biết.
- 26.3 dùng SDL3; renderer shaders được biên dịch bằng ShaderC cho cả OpenGL và Vulkan; terrain shader resources dùng GLSL 330 và explicit locations.
- Windows 10 Home build19045, Intel HD Graphics/driver 2012, 4GB RAM, Intel Core i5 M460 2-core/4-thread, no AVX.
- Probe DXGI/D3D12 đo được Microsoft Basic Render Driver (WARP); D3D12 device tạo được ở feature level 12.1.
- WGL probe hiện chỉ thấy Microsoft GDI Generic GL1.1; không nhận Intel ICD. Vulkan loader không load được; Win32 error 126. Không có vulkaninfo.
- Không có Mesa WGL DLL, SwiftShader ICD, Vulkan loader khả dụng hay Mesa build toolchain hiện hữu. CMake, Meson và Ninja không tìm thấy.

## Ứng viên

| Ứng viên/chuỗi API | Đánh giá kỹ thuật | OS/runtime | Khả năng với game 26.3 | Trạng thái |
|---|---|---|---|---|
| **Ưu tiên PoC: Minecraft OpenGL → Mesa Gallium D3D12 → D3D12 WARP** | Chuỗi hợp lệ: Mesa D3D12 phát D3D12 thay vì target GPU cụ thể; Mesa công bố full desktop OpenGL 3.3 support qua driver D3D12. WARP là D3D12 software adapter. Máy đã đo có WARP feature level12.1. GLSL 330 trong game phù hợp baseline GL3.3, nhưng extension/function coverage, FBO, ShaderC output, SDL3/WGL và selection của game phải test. | Windows 10 x64; cần Mesa D3D12 WGL/OpenGL runtime DLLs tương thích x64 và chọn adapter Microsoft Basic Render Driver bằng MESA_D3D12_DEFAULT_ADAPTER_NAME. Driver chạy trên CPU; không phải GPU acceleration. | Đường trực tiếp nhất để phục vụ desktop OpenGL cho GL backend và tránh native OpenGL. GL3.3 không mặc nhiên đáp ứng GL4.x fast paths; những fast paths/fallback trong JAR phải được test. Mod renderer vẫn chưa có để đánh giá. | **Ứng viên khả thi về API, chưa tích hợp.** Mesa build/binary không có; chưa tạo test instance. |
| **SwiftShader: Minecraft Vulkan → LWJGL/Vulkan loader → SwiftShader ICD (CPU)** | SwiftShader chính thức là Vulkan 1.3 CPU implementation và có build Windows. Game JAR có Vulkan backend, LWJGL Vulkan3.4.3/VMA. Cần xác minh Vulkan extensions/features và SDL3 Win32 surface tương ứng với requiredFeatureSets trong bytecode. | Windows x64; cần đúng loader/ICD DLL và cấu hình per-test process. Probe hiện không load được system vulkan-1.dll; SwiftShader chưa có. | Bỏ qua native OpenGL, nhưng dùng Vulkan backend của game. Có thể giữ gameplay Java; tương thích mod renderer không chắc. RAM 4GB/CPU 2-core làm software Vulkan rủi ro cao. | **Chưa thử; ứng viên dự phòng.** Binaries và loader chưa có. |
| Mesa LLVMpipe: Minecraft OpenGL → Mesa WGL/OpenGL → LLVMpipe CPU | Mesa ghi rõ Windows có thể build app-local opengl32.dll + libgallium_wgl.dll. Đây là desktop GL software path thực, nhưng mức GL/extensions cần xác minh ở build cụ thể. | Windows x64 build cần Mesa/LLVM toolchain. Tại máy này không có Mesa DLL, CMake/Meson/Ninja không tìm thấy. | SDL3/WGL integration trực tiếp hơn ANGLE; CPU rasterization có thể chậm. CPU có SSE4.1 nhưng chỉ 2 lõi và 4GB RAM. | **Có cơ sở kỹ thuật, build chưa khả thi trong môi trường hiện tại.** |
| Mesa Zink: Minecraft OpenGL → Zink → Vulkan ICD/GPU hoặc software ICD | Zink là OpenGL-over-Vulkan. GL2.1 tối thiểu đã cần Vulkan1.0 + feature/extensions; GL3.3 đòi thêm D3D-like device features, vertex attribute divisor, format/limit requirements. Minecraft shader sources cần GL3.3 semantics. Zink không biến Vulkan yếu/không có thành GPU hiện đại. | Cần Mesa Zink build và Vulkan ICD. Không có Vulkan loader/ICD phát hiện. | Kỹ thuật hợp lệ nếu Vulkan backend phù hợp, nhưng thêm một lớp dịch so với Minecraft Vulkan backend trực tiếp. Không có bằng chứng device đạt profile Zink3.3. | **Không ưu tiên; prerequisite Vulkan hiện không đo được.** |
| ANGLE: desktop OpenGL game → ??? → ANGLE GLES → D3D/Vulkan | ANGLE repo mô tả conformant OpenGL ES implementation. Minecraft backend cung cấp desktop OpenGL, không phải GLES. Chưa tìm thấy adapter desktop GL→ANGLE GLES cho SDL3/LWJGL trong artifacts. | Windows binaries/dependencies chưa có. | Không phải drop-in desktop GL replacement; yêu cầu port/adaptation không có bằng chứng. | **Không phù hợp như drop-in.** |
| Microsoft WARP trực tiếp | WARP là Direct3D software rasterizer, không có OpenGL API đầu vào. Tuy nhiên chuỗi Mesa D3D12 → WARP ở dòng đầu là đường dịch hợp lệ có bằng chứng. | D3D12 WARP hiện đã đo được feature level12.1. | WARP đơn lẻ không chạy Minecraft GL. Mesa D3D12 wrapper cần có và chưa được cài/build. | **Có trong chain Mesa D3D12; không phải wrapper độc lập.** |
| Native Vulkan Intel HD | Game có Vulkan backend nhưng Vulkan loader không load được trong probe, chưa có ICD/device feature list; driver GPU từ 2012. | Chưa xác minh Vulkan support. | Không thể đánh giá suitability chỉ từ dxdiag DirectX12. Native D3D12/WARP không chứng minh Intel GPU có Vulkan. | **Chưa xác minh; không chọn để PoC hiện tại.** |

Nguồn: Mesa D3D12 https://docs.mesa3d.org/drivers/d3d12.html ; Mesa driver overview https://docs.mesa3d.org/systems.html ; Microsoft WARP adapter https://learn.microsoft.com/en-us/windows/win32/api/dxgi1_4/nf-dxgi1_4-idxgifactory4-enumwarpadapter ; SwiftShader https://github.com/google/swiftshader/blob/master/README.md ; Mesa Zink https://docs.mesa3d.org/drivers/zink.html ; LLVMpipe Windows https://docs.mesa3d.org/drivers/llvmpipe.html ; ANGLE https://github.com/google/angle

## Đánh giá ưu tiên

### 1) Mesa D3D12 trên WARP — PoC đầu tiên nên thử

**Lý do:** đây là đường duy nhất đã nối được API mà OpenGL backend cần tới một software API backend đã đo được trên máy: OpenGL 3.3 từ Mesa D3D12 → D3D12 → WARP feature level12.1. Mesa mô tả D3D12 driver là desktop OpenGL3.3 trên D3D12. Minecraft GLSL 330 sources đặt mức ngôn ngữ đúng sát GL3.3. Không cần Vulkan ICD trên máy.

**Chưa được xác minh:** shaderc/SPVC GL output, GL_ARB_separate_shader_objects, FBO/format limits, persistent buffers, SDL3 context và presentation, backend setting, rendering gameplay, mod hooks. D3D12 WARP chạy software; hai lõi CPU/4GB RAM thấp hơn Minecraft.net minimum target, nên hiệu năng có thể không dùng được. Không ước lượng FPS khi chưa benchmark.

### 2) PoC an toàn khi có test path và Mesa runtime

1. Giữ instance 26.3 gốc bất biến; test trên instance copy riêng.
2. Dùng Mesa D3D12 WGL DLLs từ build có nguồn/version/hash đã xác minh, đặt cạnh một Java runtime copy riêng, không thay Windows/system DLL hay Java runtime dùng chung.
3. Chọn adapter software bằng biến môi trường MESA_D3D12_DEFAULT_ADAPTER_NAME=Microsoft Basic Render Driver cho riêng tiến trình test; chọn OpenGL qua graphics API option trong bản sao.
4. Lưu launcher config/options trước-sau; rollback bằng cách bỏ instance test và các DLL cục bộ.
5. Kiểm tra tuần tự: WGL context/version/extensions → ShaderC + shader compile/link → menu → world → camera/chunk → entity/particles/transparency/OIT off/on → mod từng nhóm → benchmark 854x480, warm-up và 60 giây.
6. Ghi console/log, adapter name và GL/GLSL versions; không coi menu là test gameplay.

### 3) SwiftShader dự phòng

Dùng riêng test instance và official Windows ICD/DLLs; phải kiểm tra backend Vulkan enumerates đúng device, surface/presentation, requiredFeatureSets, shader pipeline, world render và mod matrix. Không thêm SwiftShader vào native driver/global registry. Performance phải đo; CPU-based Vulkan không bảo đảm FPS.

## Rollback và trạng thái

Chưa tạo PoC vì không có thư mục test ngoài workspace được phép ghi, không có Mesa/SwiftShader binary, và không có runtime C++/Mesa build tools. Không sửa setting hay biến môi trường toàn cục. Nếu thực hiện sau: chỉ thêm file vào bản sao/test runtime; rollback bằng cách xóa bản sao và DLL trong sandbox cho phép, giữ nguyên 26.3 gốc.
