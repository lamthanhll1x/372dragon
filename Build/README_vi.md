# Build — đầu ra dựng (không đưa vào git)

## Build nhanh (Windows, VS 2022 Build Tools)

Cấu hình đầy đủ: `Docs/12-CAU-HINH-BUILD-CLIENT_vi.md`.

```bat
rem Client (Game.exe + Render.dll + UI_CEGUI.dll), cây build riêng:
Tools\build\build.cmd x64 client-x64 Release "zlib;zziplib;freetype;devil;ogremain;ogre_d3d9;ogre_plugins;opcode;tinyxml;expat;scorpio;particlefx2;wxengine;wxcore;wxrender;pathlib;cegui;falagardex;luaplus;uisystem;wxnetpackets;protocolgen;netcore;wxclient"
rem Chỉ Game.exe (nhanh; nhớ rằng Render/UI là DLL nạp lúc chạy):
Tools\build\build.cmd x64 coord-x64 Release "" cys_wxclient
rem Toàn bộ 33 thành phần (như CI):
Tools\build\build.cmd x64 audit-clean Release
```

Đúng thứ tự tham số: `build.cmd ARCH NAME CONFIG "components" TARGET` — **không** đặt tên target
vào vị trí CONFIG (build.cmd sẽ từ chối). Hoặc dùng preset trong `Src\CMakePresets.json`
(`cmake --preset client-x64` rồi `cmake --build --preset client-x64`) từ prompt đã có MSVC.
Sau đó ghép bản chạy: `Tools\build\stage.ps1` → `Build\Client\Bin64`.

| Thư mục | Nội dung |
|---|---|
| `Client\Bin64\`, `Client\Bin\` | Game.exe và các DLL dựng từ `Src`, cộng runtime của hãng (VC++, D3DX9_43, FMOD, cg), theo đúng bố cục thư mục game |
| `Client\Data\` | Dữ liệu dạng thư mục rời khi phát triển (bản sao hoặc liên kết tới `Assets\Canonical`) |
| `Server\Sandbox085\` | Hộp cát: server C++ 0.85 chép từ `Reference\…\Build\Server`, dùng cho pha E1 (`Docs\04`, mục 7). Server sản phẩm vẫn là skynet trong WSL |
| `H0\` | Bản sao `client_GL` dành cho thí nghiệm tuỳ chọn H0 (`Docs\04`, mục 10) |
| `Packs\` | Gói AXP v1.1 để phát hành |
| `Symbols\` | PDB theo cấu hình dựng |
| `Logs\<ngày>-<cổng/đợt>\` | Hồ sơ bằng chứng của từng cổng nghiệm thu hoặc đợt nhập: log, ảnh, báo cáo validator |
