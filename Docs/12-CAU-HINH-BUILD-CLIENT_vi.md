# 12 — Cấu hình build client (phân tích hiện trạng + bổ sung)

Ngày lập: 28/10/2026. Áp dụng cho cây `Src/` (CMake + Ninja + MSVC, Windows x64/x86).
Tài liệu này tổng hợp cấu hình build client từ mã thực tế và **ghi rõ những gì mình đã bổ sung**
vì trước đó thiếu (xem mục 7). Đối chiếu chiều sâu với kiểm toán source/build: `Docs/10`.

---

## 1. Kiến trúc hệ build

```text
Src/CMakeLists.txt          # điểm vào: khai báo danh sách thành phần + vòng lặp nạp
Src/cmake/CYSCommon.cmake   # thiết lập chung: C++14, CRT, cờ biên dịch/link, SDK nhập môn
Src/cmake/components/*.cmake# MỘT file / MỘT thành phần; chưa có file => bị bỏ qua (song song hoá)
Tools/build/build.cmd       # wrapper: vcvarsall -> cmake configure -> ninja build
Tools/build/stage.ps1       # ghép bin/ + runtime hãng + dữ liệu -> Build\Client\Bin64 chạy được
Tools/build/ci.ps1|ci_all   # CI: một cây ci-x64, tất cả thành phần, log + manifest + summary
```

Nguyên tắc thiết kế (đọc từ comment trong `Src/CMakeLists.txt`):
mỗi thành phần do đúng một người/agent sở hữu; thứ tự trong `CYS_ALL_COMPONENTS` = thứ tự phụ thuộc;
thành phần nào chưa có file `.cmake` thì CMake in `STATUS ... (bỏ qua)` và đi tiếp — nhờ vậy các
agent làm song song không chặn nhau. Cơ chế này **đang hoạt động đúng**: đối chiếu ngày 28/10/2026,
cả 33 tên trong `CYS_ALL_COMPONENTS` đều có file tương ứng trong `Src/cmake/components/`,
không có file mồ côi (không được khai báo).

## 2. Cấu hình chung (`Src/cmake/CYSCommon.cmake`)

| Mục | Giá trị | Ghi chú |
|---|---|---|
| Standard | `CMAKE_CXX_STANDARD 14`, `CXX_STANDARD_REQUIRED ON` | toàn cây |
| CRT | `MultiThreaded$<$<CONFIG:Debug>:Debug>DLL` → `/MD`, `/MDd` | khớp VC project gốc (MultiByte, /MD) |
| PIC | OFF | mặc định CMake trên Windows |
| Output | `<tree>/bin` (EXE/DLL/PDB), `<tree>/lib` (.lib) | **không** ghi thẳng vào `Build/Client/Bin64` để nhiều cây build song song không đè nhau; bước *stage* mới chép |
| Arch | `CYS_ARCH` = `x64`/`x86` theo `CMAKE_SIZEOF_VOID_P` | dùng chọn thư viện DX9 |
| Encoding | `/source-charset:utf-8 /execution-charset:.936` | literal tiếng Trung là **khoá tài nguyên** → sinh byte GBK |
| Warnings | `/W3 /w34701 /w34703 /w34715` + danh sách `/wd…` (4244, 4267, 4018, …) | không `/WX`; nâng cấp mức cảnh báo là tính năng (CYS-WARNSWEEP 19/09). Đừng lên `/W4` |
| Defines | `_CRT_SECURE_NO_WARNINGS`, `_HAS_AUTO_PTR_ETC=1`, `_SILENCE_*`, `WIN32 _WINDOWS`, `NDEBUG` khi không Debug | |
| Link | `/DEBUG` + `/OPT:REF /OPT:ICF` (non-Debug) | PDB luôn sinh |
| Parallel | `/MP` | cộng `CMAKE_BUILD_PARALLEL_LEVEL` ở CI |

SDK/nhập môn:
- **DirectX 9**: `ThirdParty/DirectX9/Include` + `Lib/{x86,x64}` (`CYS_DX_INC/CYS_DX_LIB`).
  Riêng WXClient **không** được đặt include DX9 lên `/I` vì `rpcsal.h/specstrings` của SDK cũ
  che bản Windows SDK (gây C2061); `dinput8/dxguid` lấy từ Windows SDK.
- **boost**: header-only, target INTERFACE `cys_boost`; `BOOST_ALL_NO_LIB` đã bật trong
  `boost/config/user.hpp`.
- **FMOD Core 2.02.35**: tuỳ chọn, nằm ở `ThirdParty/_sdk/fmod/api/core` (**không commit**,
  `.gitignore:23`). Có header+lib ⇒ imported target `cys_fmod` + `CYS_HAVE_FMOD=1` + POST_BUILD
  chép `fmod.dll` cạnh Game.exe; không có ⇒ sound là stub câm (cấu hình vẫn pass).
- **Cg 3.1**: chỉ cần cho `Plugin_CgProgramManager`; import lib **sinh lúc configure** bằng
  `link /dump /exports` + `lib.exe` từ một `cg.dll` x64 (`-DCYS_CG_DLL=…`, mặc định
  `${CYS_ROOT}/../client_GL/Bin64/cg.dll`). Không tìm thấy ⇒ target bị bỏ qua (log STATUS).
  CI (`ci.ps1`) đã biết truyền `-DCYS_CG_DLL`.

## 3. Chọn thành phần & cây build

```bat
Tools\build\build.cmd <ARCH> <NAME> [CONFIG] ["comp1;comp2;..."] [TARGET]
rem   ARCH   : x64 | x86          NAME: thư mục con Build\cmake\<NAME>
rem   CONFIG : Release(mặc định) | Debug | RelWithDebInfo | MinSizeRel
rem Ví dụ — chỉ Game.exe (Release đúng nghĩa):
Tools\build\build.cmd x64 coord-x64 Release "" cys_wxclient
rem Cả cây, thư mục build riêng:
Tools\build\build.cmd x64 audit-clean Release
rem Client đầy đủ (Game.exe + Render.dll + UI_CEGUI.dll), không dựng viewer/test/chẩn đoán:
Tools\build\build.cmd x64 client-x64 Release "zlib;zziplib;freetype;devil;ogremain;ogre_d3d9;ogre_plugins;opcode;tinyxml;expat;scorpio;particlefx2;wxengine;wxcore;wxrender;pathlib;cegui;falagardex;luaplus;uisystem;wxnetpackets;protocolgen;netcore;wxclient"
```

Các biến cache đáng nhớ:

| Biến | Mặc định | Ý nghĩa |
|---|---|---|
| `CYS_COMPONENTS` | tất cả 33 | danh sách nạp; CI dùng `-UCYS_COMPONENTS` để xoá cache cũ |
| `CYS_CG_DLL` | `…/client_GL/Bin64/cg.dll` | nguồn cg.dll x64 cho Plugin_CgProgramManager |
| `CYS_OGRE_LIB_DIR` | `Build/cmake/ogre-<arch>/lib` | đường lùi: link `OgreMain.lib` của cây khác khi `ogremain` không ở cùng tree |
| `CYS_OGRE_WARN_TRUNC` | OFF | bật lại C4244/C4267 trên target OGRE (săn truncation x64) |
| `CYS_PARTICLES_UNIT` | ON | tắt trong cây benchmark (`Tools/perf/build_particle_bench.cmd`) |
| `CYS_PG_ASAN` | OFF | ASan cho `cys_protocolgen` + tests (cây riêng) |
| `CMAKE_PROJECT_CYSource_INCLUDE` | – | cơ chế “inject” mà checks/perf/fx test dùng để thêm target phụ |

Bẫy đã sửa (CYS-BUILDCFG 18/09): từng có thói quen gọi
`build.cmd x64 NAME cys_wxclient` ⇒ tên target rơi vào `CMAKE_BUILD_TYPE` ⇒ **không có cờ tối ưu**
mà build vẫn pass. `build.cmd` nay **từ chối CONFIG lạ** (rc=2, kèm hướng dẫn). Đúng là
`build.cmd x64 NAME Release "" cys_wxclient`.

## 4. Bản đồ thành phần → artefact (client)

Target thật trong CMake ≠ tên file phát hành (`OUTPUT_NAME`):

| Thành phần | Target | Artefact | Loại |
|---|---|---|---|
| wxclient | `cys_wxclient` | **Game.exe** | WIN32 EXE |
| wxengine | `cys_wxengine` | WXEngine.**lib** | STATIC (tĩnh vào Game.exe) |
| wxcore | `cys_wxcore` | WXCore.**lib** | STATIC |
| wxnetpackets | `cys_wxnetpackets` | WXNetPackets.lib | STATIC |
| netcore / protocolgen | `cys_netcore` / `cys_protocolgen` | .lib | STATIC (netcore **FATAL_ERROR** nếu thiếu protocolgen) |
| pathlib | `cys_pathlib` | PathLib.dll | SHARED (clean-room navmesh) |
| luaplus | `cys_luaplus` | LuaPlus.dll | SHARED; `LUAPLUS_HAS_WCHAR_T` PUBLIC — phải khớp giữa DLL và Game.exe |
| wxrender | `cys_wxrender` | **Render.dll** | SHARED plugin (nạp lúc chạy) |
| uisystem | `cys_uisystem` | **UI_CEGUI.dll** | SHARED plugin (def exports DllInit/DllMagicCode/DllRelease) |
| cegui | `cys_ceguibase/falagard/ogrerenderer` | CEGUIBase.dll, CEGUIFalagard.dll, CEGUIOgreRenderer.dll | SHARED |
| falagardex | `cys_falagardex` | CEGUIFalagardEx.lib | STATIC, được **ghép vào CEGUIFalagard.dll** |
| ogremain / ogre_d3d9 / ogre_plugins | `OgreMain`, `RenderSystem_Direct3D9`, `Plugin_*` | DLL | engine |
| particlefx2 / scorpio | `cys_particlefx2`, `cys_scorpio` | Plugin_ParticleFX2.dll, Plugin_Scorpio.dll | PREFIX "" |
| zlib zziplib freetype devil opcode tinyxml expat | `cys_*` | .lib | third-party tĩnh |
| viewer, ogre_smoke, ui_viewer, netbot, postfx_test, shadow_test, actionset_check, d3d9trace | EXE/DLL chẩn đoán | **không stage** vào Play | Tools |

Điểm ngữ nghĩa quan trọng: **Game.exe không link Render.dll/UI_CEGUI.dll** — nạp lúc chạy bằng
`tKernel::LoadPlugin` + class registry (`NewNode("CRenderSystem"/"CUISystem"/…)`, xem
`GameProcedure.cpp:635`, `TDKernel.cpp:155`). Vì vậy build riêng target `cys_wxclient` **không bảo đảm**
DLL render/UI đã dựng lại; muốn chắc phải dựng cả nhóm hoặc dùng CI + stage.

Ràng buộc ngầm khi chọn subset thủ công (CMake chỉ WARN, không fail):
- `wxclient`: thiếu `pathlib`/`luaplus` ⇒ biên dịch xong nhưng **không link**; thiếu `netcore` ⇒ mất
  transport đăng nhập hiện đại (`CYS_HAVE_NETCORE`); thiếu `protocolgen` mà có `netcore` ⇒ FATAL.
- `uisystem`: cần `cegui` + `falagardex` (headers) + `luaplus` + `wxengine` + `ogremain`
  (hoặc `CYS_OGRE_LIB_DIR` trỏ tới OgreMain.lib dựng sẵn).
- `viewer`: cần `ogremain` + `wxcore` cùng tree, nếu không thì bỏ qua (WARN).
- `LuaApiGen` được GLOB vào Game.exe ⇒ bật `CYS_HAVE_LUAAPIGEN=1`; `Game.rc` biên dịch bằng
  custom command `rc /l 0x804` (GBK, `#pragma code_page(936)`) để cờ C/C++ chung không lọt sang rc.exe.

## 5. Từ `bin/` tới bản chạy — stage

`Tools/build/stage.ps1` (mặc định Tree=`Build\cmake\ci-x64`, Out=`Build\Client\Bin64`):
- Chép **mọi** `*.dll/*.exe` trong `<tree>\bin` (Game.exe, viewer, test app, OgreMain,
  RenderSystem_Direct3D9, plugins, `cg.dll` do POST_BUILD chép vào; fallback đọc `CYS_CG_DLL` từ cache).
- Chép runtime của hãng: `d3dx9_43.dll`, `D3DCompiler_43.dll` từ System32; `fmod.dll`, `fmodL.dll` từ FMOD SDK.
- Chép `Engine/Scorpio/media` → `Bin64\Media\Scorpio`; PDB → `Build\Symbols\ci`.
- **Không copy dữ liệu game**: dùng read-only `Assets\CanonicalCY\Data` qua `--data`
  (junction `Data` ở bản Play); lựa chọn + hash ghi trong `Bin64\cys_stage.json`.
- Exit codes: 0 ok, 1 ok-có-vấn-đề (thiếu runtime/file không phải x64), 2 lỗi, 3 CI bận.
- Tuyệt đối không copy đệ quy cả `build/bin` vào Play (có tool chẩn đoán không dùng khi chơi).

## 6. CI / kiểm chứng

- `Tools/build/ci.ps1`: một cây `Build\cmake\ci-x64`, `-UCYS_COMPONENTS` (all), Ninja + vcvarsall x64,
  `-Config`, `-Jobs`, File API codemodel để summary liệt kê targets/artefacts; log + metadata JSON
  (configure rc/giây, build rc/giây, sha, dirty, fmod_visible, cg_dll…). `-Fresh` cách ly cây cũ.
  Build: `cmake --build <tree> --parallel N -- -k 0` (keep going, báo mọi lỗi).
- `checks.ps1`: dựng cây kèm inject `Tests\scene\scene_test.cmake` + `-DCYS_CG_DLL`.
- `tests.ps1`, `smoke.ps1` (`_NT_SYMBOL_PATH` → Build\Symbols), `dll_cluster_check.py`,
  `parse_projects.py`+`projects.json` (đối chiếu VC project gốc ↔ CMake), `ci_summary.py`, `ci_timings.ps1`.
- Toolchain đã kiểm chứng: VS 2022 Build Tools, MSVC 14.44.35207, CMake/Ninja đi kèm, x64 Release
  (`Docs/10` mục 8). `build.cmd` hardcode `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools`
  — máy khác vị trí cài thì phải sửa biến `VS`.

## 7. Khoảng trống tìm được và đã bổ sung

| # | Hiện trạng trước | Đã bổ sung |
|---|---|---|
| 1 | Không có tài liệu mô tả **toàn bộ** cấu hình build client (thông tin rải trong comment CMake + Docs/10) | Tài liệu này (`Docs/12`) |
| 2 | `Build/README_vi.md` chỉ nói về đầu ra, không có lệnh build; người mới dễ gọi sai `build.cmd` (chính cái bẫy CONFIG ở mục 3) | Thêm mục “Build nhanh” trong `Build/README_vi.md` |
| 3 | Không có preset tái lập cấu hình (component list của client phải gõ tay, rất dài) | `Src/CMakePresets.json` — preset `client-x64` (all client components) và `game-exe-x64`; giữ nguyên `build.cmd`/CI, preset chỉ là đường tắt |
| 4 | Danh sách 33 component không có sơ đồ phụ thuộc để tra cứu | Mục 4 ở trên + bảng target↔file |

Không thay đổi bất kỳ logic build nào (không sửa `CYSCommon.cmake`, không sửa component files),
vì cấu hình hiện tại đã được CI và kiểm toán xác nhận; chỉ bổ sung tài liệu + preset.

## 8. Checklist khi thêm/sửa một thành phần

1. Tạo/sửa đúng MỘT file `Src/cmake/components/<tên>.cmake`; thêm tên vào `CYS_ALL_COMPONENTS`
   theo **thứ tự phụ thuộc** (library trước, consumer sau). Tên chưa có file ⇒ được bỏ qua an toàn.
2. Dùng global flags của `CYSCommon.cmake`; chỉ thêm cờ đặc thù trong file của mình (như `/EHa`
   cho `WXClient.cpp`, `/bigobj` cho wxclient).
3. Dependency qua **target CMake tường minh**, không `#pragma comment(lib)`; nếu optional thì
   `if(TARGET …)` + `message(WARNING …)` theo khuôn mẫu wxclient.
4. Đặt `OUTPUT_NAME` = tên file phát hành, `PREFIX ""` cho plugin OGRE/tEngine, `FOLDER` cho IDE.
5. Runtime DLL bên ngoài (cg/fmod) ⇒ POST_BUILD `copy_if_different` vào `$<TARGET_FILE_DIR:…>`
   để stage nhìn thấy trong `bin/`.
6. Kiểm chứng: `build.cmd x64 <name> Release "<subset>"` → `ci.ps1` (all) → `stage.ps1` → `smoke.ps1`;
   đối chiếu PE bằng `Tools/audit/audit_client_binaries.py`.
