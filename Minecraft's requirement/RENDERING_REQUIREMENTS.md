# Yêu cầu rendering Minecraft Java 26.3 và giới hạn OpenGL 2.1

Ngày khảo sát: 2026-10-28  
Bằng chứng tại chỗ: shared metadata 26.3, client JAR 26.3 đã đối chiếu SHA-1, 89 shader resources, class/constant references của backend OpenGL/Vulkan. Không có source project hoặc log game.

## Kết luận

**Đã xác minh:** Client 26.3 chứa cả backend OpenGL và Vulkan, SDL3 cho window/input/platform integration, ShaderC cho shader khi dùng cả hai backend, và code dùng OpenGL có references tới GL 3.3 APIs cùng các extension/fast paths đời mới. Không có options.txt hay game log để biết backend đã chọn trong instance.

**Mismatch quan trọng với native GL 2.1:** JAR chứa terrain.vsh/terrain.fsh và OIT shaders dùng #version 330, explicit layout(location=...) và yêu cầu GL_ARB_separate_shader_objects. OpenGL 2.1 chỉ đảm bảo GLSL 1.20. Đây là mismatch shader rõ ràng nếu shader đi trực tiếp tới driver GL 2.1. ShaderC/SPVC có mặt, nhưng không tìm thấy bằng chứng rằng GL backend hạ các shader này xuống GLSL 1.20 hoặc chuyển ngược các API hiện đại sang GL 2.1. Vì vậy khả năng chạy native GL 2.1 không được xem là khả thi đã chứng minh; cần một backend software/translation cung cấp tối thiểu các semantics GL phù hợp. Khronos mapping/spec: https://registry.khronos.org/OpenGL/index_gl.php

Không suy ra mọi tính năng hiện đại đều bắt buộc. Release notes gọi MultiDrawIndirect là dùng trên thiết bị hỗ trợ; class tree cũng có các implementation mang tên Emulated/Fallback cho một số chức năng. Bảng dưới đánh dấu phần chưa truy vết là chưa xác minh.

## Pipeline quan sát được

1. Application: client JAR có GameRenderer, RenderSystem, com.mojang.renderpearl.backend.opengl và com.mojang.renderpearl.backend.vulkan. Options.class có preferredGraphicsBackend/preferredGraphicsBackendFromStartup và graphicsApi option. Không có options.txt/log hiện hành nên backend thực được chọn chưa biết.
2. Window/input: release notes 26.3 xác nhận SDL3 thay GLFW. JAR có SDLEventHandler, Window bytecode tham chiếu org.lwjgl.sdl và SDL_GL_GetProcAddress/SDL_GL_SetAttribute. Cache có lwjgl-sdl-3.4.3.jar và SDL native Windows x64. Wrapper phải bảo toàn SDL window, GL context hoặc Vulkan surface và đường tải hàm.
3. Shader: release notes xác nhận ShaderC dùng cho OpenGL lẫn Vulkan; include dùng #include; inputs/outputs phải có explicit location. Trong JAR có lwjgl-shaderc 3.4.3 và natives, lwjgl-spvc 3.4.3 và natives.
4. Geometry/draw: backend OpenGL bytecode tham chiếu GL33C, ARBBaseInstance, ARBDrawIndirect, ARBMultiDrawIndirect, ARBVertexAttribBinding. Release notes nói terrain dùng MultiDrawIndirect trên thiết bị hỗ trợ.
5. Render targets/effects: JAR có FrameBufferCache, GlRenderPass, OitPipelineSet/OitRenderPassProvider, post-effect JSON và OIT shader resources. Không có game trace để xác định attachment format, attachment count, hay pass nào đang hoạt động trong cấu hình cụ thể.
6. Resource synchronization: backend có GlFence, GlTransientMemory persistent-mapping/fallback, BufferStorage immutable/mutable. API được gọi có thể capability-gated; chưa truy vết toàn bộ điều kiện lựa chọn.
7. Mods: không có mod JAR để khảo sát hook, mixin, native/JNI hoặc API dependency.

Nguồn release notes chính thức: https://www.minecraft.net/en-us/article/minecraft-java-edition-26-3

## Ma trận OpenGL

| Feature | GL core / extension | Bằng chứng trong 26.3 | Phân loại | Tác động native GL 2.1 |
|---|---|---|---|---|
| GLSL cơ bản | GL 2.1 đảm bảo GLSL 1.20 | Khronos spec; game terrain resources dùng #version 330 | Shader 3.30 là đường game được đóng gói; chưa chứng minh có hạ xuống GLSL 1.20 | Không tương thích trực tiếp theo dialect. GL 3.3 hoặc extension/translator đủ semantics cần thiết là mức thử hợp lý |
| Shader inputs/outputs location | Core GLSL 3.30 và extension liên quan | terrain.vsh/fsh có layout(location); ShaderC thay đổi location trong release notes | Dùng trong shaders; fallback/down-translation chưa thấy | GL 2.1/GLSL1.20 không đáp ứng mặc định |
| ShaderC và include | Compiler library, không tự cung cấp driver API | Release notes; LWJGL shaderc/SPVC 3.4.3; shader dùng #include | Đã xác minh compiler flow tồn tại; target GLSL/translation cuối chưa truy vết | ShaderC không tự thêm GL functions, framebuffer formats hoặc context capabilities |
| VBO/IBO, shader attributes | Buffer objects core GL 1.5; vertex shader/attributes core GL 2.0 | GL backend có bindBuffer/bufferData, vertex attribute calls | Được bytecode tham chiếu; GL 2.1 có nền tảng cơ bản | Một phần tương thích; không giải quyết shader 3.30 và API sau GL2.1 |
| FBO/off-screen | Core GL 3.0; extension ARB/EXT có thể có trên GL2.1 | FrameBufferCache và GL framebuffer/blit calls | Backend có dùng; đường EXT/ARB cho 2.1 và fallback chưa xác minh | Cần kiểm tra extension/semantics thực, đặc biệt post effects/OIT |
| VAO | Core GL 3.0 hoặc ARB_vertex_array_object | VertexArray$Emulated tồn tại cùng Separate implementation | Có implementation emulated, cho thấy VAO fast path không nhất thiết bắt buộc | Có thể có fallback; không khẳng định GL2.1 qua được shader/API còn lại |
| UBO | Core GL 3.1 hoặc ARB_uniform_buffer_object | Uniform$Ubo và calls bindBufferRange/getUniformBlockIndex | Code path tồn tại; điều kiện chọn và fallback chưa truy vết | Thiếu core GL2.1; cần extension hoặc fallback thực |
| Instancing | GL 3.1 / ARB_draw_instanced; base instance GL4.2 / ARB_base_instance | GL backend có instanced calls và ARBBaseInstance | Fast path/capability gating chưa truy vết đầy đủ | GL2.1 thiếu core; mod dùng trực tiếp sẽ nhạy |
| Sync/fences | GL 3.2 / ARB_sync | GL backend tham chiếu glFenceSync, glClientWaitSync, glDeleteSync | Sử dụng trong backend; fallback hoàn chỉnh chưa xác minh | Thiếu core GL2.1; có thể chặn resource lifetime nếu bắt buộc |
| Persistent mapped buffers | GL4.4 / ARB_buffer_storage | PersistentMapping, Fallback, BufferStorage.Immutable/Mutable; ARBBufferStorage refs | Có fallback class cho transient memory/storage; các điều kiện capability chưa truy vết hết | Có đường fallback tồn tại ở mức class, mức đầy đủ chưa kiểm thử |
| Direct State Access | GL4.5 / ARB_direct_state_access | DirectStateAccess$Core và DirectStateAccess$Emulated | Có emulated path | Không bắt buộc nếu emulated path được chọn; cần runtime test |
| Texture arrays | Core GL3.0 / EXT_texture_array | Không tìm thấy bằng chứng cụ thể trong shader/core method refs đã quét | Chưa xác minh có dùng | Không gắn kết luận vào yêu cầu game |
| Texture buffers | Core GL3.1 / ARB_texture_buffer_object | Backend GL có glTexBuffer; Uniform$Utb class tồn tại | Có API reference; mục đích cụ thể chưa xác minh | GL2.1 không có core support |
| Compute shaders | Core GL4.3 / ARB_compute_shader | Không tìm thấy GL43C compute refs hoặc shader .comp/.csh trong tập shader đã kiểm tra | Không có bằng chứng game GL backend dùng; không kết luận game không bao giờ dùng | Chưa xác minh; không gọi là hard requirement |
| MultiDrawIndirect | Core GL4.3 hoặc ARB_multi_draw_indirect | ARBMultiDrawIndirect refs trong GL backend; release notes ghi “on supported devices” | Optional/capability-gated theo release notes; fallback cụ thể chưa disassemble | GL2.1 không có core; terrain path có thể chọn draw path khác |
| Depth/stencil, blend | API cơ bản có từ OpenGL đời cũ; FBO attachment behavior phụ thuộc core/extension | Renderer dùng depth/blend/FBO paths; log không có thông số | Một phần đã xác minh; format/limits chưa đo | GL2.1 có depth/blend cơ bản; off-screen paths cần FBO support |
| OIT/transparency | Shader/passes cụ thể của renderer; không tương đương một extension duy nhất | OIT shaders trong JAR dùng #version330; release notes cho biết Improved Transparency bật OIT; khi tắt behavior gần như không đổi và vẫn sort geometry | OIT là lựa chọn video optional; exact FBO/MRT/float-format requirements chưa truy vết | Tắt OIT tránh path OIT nhưng không tránh shader terrain 3.30 |
| Post effects | Render targets và post-effect pipelines | JAR có post_effect JSON; resource pack có thể khai báo end_of_frame | Tính năng post effects được hỗ trợ; resource pack instance không có | FBO/formats cụ thể chưa đo |
| Texture units/formats/sRGB/max sizes | Phụ thuộc thiết bị, core và extension | Không có log game hoặc API trace. GL probe hiện chỉ GL1.1 nên các modern limit queries không hợp lệ | Chưa xác minh | Không suy từ string version; cần test trên GL context thật |
| SDL3 context/presentation | SDL3 GL hoặc Vulkan window/surface path | 26.3 chuyển window/input sang SDL3; LWJGL native SDL cache có | Đã xác minh backend cửa sổ; context config thực tế chưa ghi log | DLL shim phải tương thích Windows WGL calls và không phá SDL window/swap/fullscreen/input |

### Đã đo về driver trong phiên hiện tại

WGL tạo context Microsoft GDI Generic OpenGL 1.1.0, GLSL string null, WGL_ARB_create_context không có. Đây không phải Intel ICD; do vậy chưa xác minh native Intel OpenGL tối đa. Đã kiểm tra extension names trong context này nhưng không có modern extension. Không dùng các số query GL_MAX_* thu được từ context 1.1 vì enum hiện đại không hợp lệ ở context đó.

## Phân loại kết luận

- **Đã xác minh:** JAR 26.3 có backend OpenGL và Vulkan; SDL3; GLSL 330 terrain/OIT resources; ShaderC changes; MDI fast path có điều kiện; game metadata yêu cầu Java25.
- **Đã đo:** WGL Generic 1.1 hiện hữu trong session; D3D12 WARP feature level 12.1.
- **Suy luận có căn cứ:** native GL2.1 không thể trực tiếp đáp ứng GLSL330 shader sources nếu không có đường hạ shader hoặc software translation. Chưa có evidence về hạ xuống GLSL1.20.
- **Chưa rõ:** backend được chọn, GL feature minimum chính xác bên trong mọi call path, Vulkan feature set đầy đủ của máy, mods, gameplay behavior và hiệu năng.
