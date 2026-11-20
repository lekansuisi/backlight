# C1 Mirror97 RGB · ZMK 固件配置包

> 一句话：这是 C1（**带 98 灯 RGB 背光**那版）的固件配方。放进 GitHub 仓库，云端自动编译出 `.uf2`，拖进主控即可。
> 键盘矩阵、键位、Fn 层的 BT/OUT/音量键**完全沿用已验证的 B1 配置**，本包只新增 RGB 灯链。

---

## 一、这套配置对应哪块板

| 项目 | 值 |
|---|---|
| 键盘 | C1 Mirror97 RGB · 97 键 · 左手真镜像数字区 · Choc V2 |
| 主控 | nice!nano v2（NRF52840） |
| ZMK board / shield | `nice_nano_v2` / `c1_mirror97` |
| ZMK 版本 | `v0.3` |
| 矩阵 | 10×10，97 键 + 3 空位，`row2col` |
| RGB 灯链 | **98 颗 SK6812 MINI-E**（97 键下灯 + 1 状态灯） |
| 数据脚 | **P1.07**（`&gpio1 7`）→ 板载 74AHCT1G125 电平转换 → 灯链 DIN |
| 驱动 | ZMK RGB underglow，`worldsemi,ws2812-spi`，SPI3 MOSI，4MHz |
| 灯序 | v20 按列蛇形重排，末灯紧邻右上角状态灯 |

---

## 二、文件夹内容

```
c1_zmk_config/
├── .github/workflows/build.yml      ← 云端编译开关（必须有，隐藏目录）
├── build.yaml                       ← 编译 nice_nano_v2 + c1_mirror97
├── config/
│   ├── west.yml                     ← 锁定 ZMK v0.3
│   ├── c1_mirror97.conf             ← 顶层配置：USB+BLE+RGB
│   └── boards/shields/c1_mirror97/
│       ├── Kconfig.shield / Kconfig.defconfig
│       ├── c1_mirror97.zmk.yml
│       ├── c1_mirror97.overlay      ← 矩阵 + RGB 灯链(SPI3/P1.07/98灯)
│       ├── c1_mirror97.conf         ← RGB Kconfig 初始值
│       └── c1_mirror97.keymap       ← 键位 + Fn 层 RGB 控制
├── zephyr/module.yml
├── 灯序映射表_v21.csv               ← 97键↔灯序号，供 per-key 模块用
├── led-indexes_keymap片段.txt        ← 可直接粘进 matrix-transform 的数组
├── 校验报告.json
└── README.md
```

---

## 三、你要做的三步（与 B1 相同）

1. **GitHub 新建空仓库**（Public，不勾 README/.gitignore），例如 `c1-zmk-config`。
2. **上传本文件夹全部内容**，务必包含隐藏的 `.github/workflows/build.yml`。
   - 命令行最稳：`cd c1_zmk_config && git init && git add -A && git commit -m "C1 RGB" && git branch -M main && git remote add origin https://github.com/你的用户名/c1-zmk-config.git && git push -u origin main`
3. **点仓库 Actions** → 等编译变绿 → 下载 Artifacts 里的 `firmware` → 解压取 `.uf2`。
   - 烧录：主控接 USB-C，0.5 秒内把 RST 碰 GND 两次进 bootloader，拖入 `.uf2`。

---

## 四、灯效控制（Fn 层 R5 一行）

按住 Fn 再按对应物理键位：

| Fn 层位置 | 功能 |
|---|---|
| `NumLock` 位 | RGB 开/关 `RGB_TOG` |
| `7` 位 | 下一个效果 `RGB_EFF` |
| `5` 位 | 上一个效果 `RGB_EFR` |
| `4` 位 | 色相 + `RGB_HUI` |
| `2` 位 | 色相 − `RGB_HUD` |
| `1` 位 | 饱和度 + `RGB_SAI` |
| `Up` 位 | 饱和度 − `RGB_SAD` |
| `0` 位 | 亮度 + `RGB_BRI` |
| `Left` 位 | 亮度 − `RGB_BRD` |

效果共 4 种：Solid 常亮 / Breathe 呼吸 / Spectrum 光谱 / Swirl 流光。默认 Solid、亮度 35%、空闲自动关灯省电。

---

## 五、两个必须知道的限制（标准 ZMK v0.3）

> 这两点不是配置写错，是 **ZMK v0.3 官方功能边界**，先说清楚避免你拿到固件后发现"和想象不一样"。

1. **整条链统一效果，按键不会单独点亮对应那颗键下灯。**
   标准 ZMK 的 RGB underglow 把 98 颗当成一条灯带，效果是整条的（呼吸/流光等）。要"按哪个键哪个键亮"的 per-key 灯效，需要第三方模块（如 `zmk-pk-underglow`）。**好消息**：我已经把 97 键→灯序号的映射算好并导出（`灯序映射表_v21.csv` + `led-indexes_keymap片段.txt`），将来接模块时直接用，不用重算。

2. **状态灯（第 98 颗）暂时随整条链显示统一颜色**，还不能单独显示"蓝牙已连 / 电量低 / Caps / Num"。
   标准 ZMK 的 LED indicator 明确不支持 WS2812 灯带。要实现状态灯单独指示，需要自定义驱动或第三方模块，属于第二阶段工作。

**本次固件的目标**：先把 98 灯点亮、效果可控、键盘正常打字——这是最稳的第一步。状态灯高级指示和 per-key 灯效可作为下一步单独做。

---

## 六、校验状态

- ✅ 静态校验全部通过：P1.07 与矩阵 GPIO 不冲突、chain-length=98、GRB 顺序、DTS 括号配平、两层各 100 绑定、RGB 枚举合法、shield 命名三处一致。
- ✅ 逐项对照 ZMK v0.3 官方源码（`ws2812-spi` overlay、`rgb.h`、`matrix-transform` binding、Kconfig）核实写法。
- ⚠️ 本地无 west/Zephyr 工具链，**未本地编译**；需 GitHub Actions 实际编译验证。若编译报错，把日志发来我修。
