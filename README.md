# Pixie Dodge

像素風彈幕生存遊戲。拖動小精靈，閃避從四面八方飛來的攻擊，撐得越久分數越高。

**線上遊玩：<https://11501-taica-hw5.elvislo.tw>**

整個遊戲只有一個 `index.html`，CSS 與 JavaScript 都寫在檔案內，不載入任何外部資源（圖片、字型、音效全部由程式產生）。完整設計見 [SPEC.md](SPEC.md)。

## 玩法

- **生存制**：沒有終點，難度隨存活時間上升，每 15 秒升一級。
- **3 條命**：被擊中扣 1 命，之後有 1.5 秒無敵時間。
- **小判定點**：只有小精靈胸前的亮點會被打中，外觀碰到子彈不算。
- **計分**：每存活 0.1 秒 +1 分；最高分保存在瀏覽器（`localStorage`）。

## 操作

| 輸入 | 動作 |
|---|---|
| 滑鼠拖曳 / 手指拖曳 | 移動（角色跟著相對位移，不會瞬移到游標下） |
| 方向鍵 / WASD | 移動 |
| 按住 Shift | 慢速精準移動 |
| P / Esc / 右上角按鈕 | 暫停 |
| M / 右上角喇叭 | 靜音切換 |

## 攻擊模式

| 解鎖時間 | 名稱 | 說明 |
|---|---|---|
| 0 秒 | Aimed | 從畫面邊緣朝玩家射擊 |
| 15 秒 | Fan | 扇形散射 |
| 30 秒 | Wall | 整排子彈推進，留一個缺口 |
| 45 秒 | Ring | 預警後爆出一整圈子彈 |
| 60 秒 | Spiral | 旋轉發射的螺旋彈幕 |
| 80 秒 | Homing | 慢速追蹤彈，4 秒後消失 |
| 100 秒 | Laser | 紅色虛線預警 1 秒後發射雷射 |

## 道具

每 12～18 秒出現一個，停留 6 秒。

| 道具 | 效果 |
|---|---|
| Shield | 抵擋 1 次傷害（最多 10 秒） |
| Bomb | 清除畫面上所有子彈 |
| Slow | 子彈速度減半 5 秒 |
| Heal | 生命 +1；滿血時改為 +50 分 |

## 本機執行

直接用瀏覽器開啟 `index.html` 即可。若要模擬正式環境，可起一個靜態伺服器：

```bash
python3 -m http.server 8000
# 開啟 http://localhost:8000
```

### 測試模式

網址加上 `#debug`（例如 `http://localhost:8000/#debug`）會在 console 開放 `window.__pixie`，方便測試，且此模式不寫入最高分：

```js
__pixie.god(true)          // 無敵
__pixie.setTime(100)       // 跳到第 100 秒
__pixie.pattern('laser')   // 立即產生指定攻擊
__pixie.item('bomb', 90, 200) // 在指定位置放道具
__pixie.info()             // 目前狀態
```

## 部署

推送到 `main` 時，GitHub Actions（[.github/workflows/deploy.yml](.github/workflows/deploy.yml)）會把 `index.html` 部署到 GitHub Pages。只有遊戲本體會上線，文件不會公開。

自訂網域 `11501-taica-hw5.elvislo.tw` 由 Cloudflare DNS 的 CNAME 指向 `elvislo030.github.io`，並在 repo 的 Settings → Pages 設定。

## 技術重點

- `<canvas>` 2D，邏輯解析度 180×320，依螢幕整數倍放大並保持像素銳利
- `requestAnimationFrame` 遊戲迴圈，以 delta time 計算移動，不同更新率的螢幕速度一致
- 子彈與粒子使用物件池，避免手機上因記憶體回收卡頓
- Pointer Events 統一處理滑鼠與觸控；切換分頁或失焦時自動暫停
- Web Audio API 即時合成 8-bit 音效
