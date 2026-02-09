# 意味・位置の分離学習による動画生成AIの長期記憶改善（Disentangled Meaning–Position Learning）

本リポジトリは、動画生成／World Model における長期記憶（長期追跡の同一性保持）を、**オブジェクト表現を「意味（identity）」と「位置（pose/geometry）」に分離して学習**することで改善する研究の実装・実験ノートブックをまとめたものです。  
**Slot Attention ベースライン**と、提案する **disentangled（意味–位置分離）モデル**を比較し、追跡安定性および長期記憶評価プロトコルにより性能を検証します。

---

## ✨ 研究のポイント（Key Idea）

長期の動画生成（ロールアウト）では、時間経過や遮蔽（occlusion）を契機に **identity drift（スロット入れ替わり／ID swap）**が生じやすく、同一物体を同一IDとして保持できなくなることが主要な課題です。  
本研究では、各オブジェクトスロットを次の2成分に明示的に分解します。

- **意味（semantic / identity）成分**：時間的に安定であるべき（同一性・外観）
- **位置（spatial / position）成分**：時間的に変化する（座標・姿勢・運動）

さらに、（任意で）外部メモリの read/write を導入し、**意味成分を安定化**することで、長期にわたる同一性保持を狙います。

---

## 📌 リポジトリ内容

- **ベースライン（Slot Attention / SAVi系の実装）**
  - `SAVi_MOVi-A_baseline_800000.ipynb`
- **提案手法（意味–位置分離モデル）**
  - `SAVi_MOVi-A_disentangled_80000.ipynb`
- **研究計画・補足資料**
  - `世界モデル研究計画.docx`

> 注：ファイル名は、比較に用いた実験スナップショット（学習ステップ等）を反映しています。

---

## ✅ 主な結果（Summary）

### Tracking 指標（同一性保持 + マスク整合）

| 設定 | モデル | ID swap (mean) ↓ | matched IoU (mean) ↑ |
|---|---|---:|---:|
| 通常評価 | Baseline | 0.3789 | 0.1667 |
| 通常評価 | Disentangled | **0.0567** | **0.5683** |
| 少数バッチ評価（batches=20） | Baseline | 0.3525 | 0.1645 |
| 少数バッチ評価（batches=20） | Disentangled | **0.0185** | **0.6759** |

- 提案法は **ID swap を大幅に低減**し、**matched IoU を大きく改善**しました。  
- 少数バッチ（batches=20）でも改善が再現され、効果の頑健性が示唆されます。

### 長期記憶評価プロトコル（Long-term memory protocol）
以下の評価も実施します：
- **ReID after occlusion**（遮蔽後の再同定）
- **Long-horizon rollout**（長期ロールアウト；MSE/PSNR）
- （任意）**Ablation gap**（評価スクリプトに含まれる場合）

現状の実験では、最大の利得は **ID swap の低減**と **matched IoU の向上**に現れており、mem/no_mem 間のロールアウト PSNR/MSE 差は非常に小さい（概ね 1e-6 オーダー）傾向です。

---

## 🧱 実行環境

### 推奨
- Python 3.10 以上
- PyTorch（GPU推奨）
- Jupyter / Google Colab

### インストール例
```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```
---
## 📁 データセット

本研究は MOVi-A 系の動画データ（SAVi/slot-based object-centric video modeling で一般的な形式）を想定しています。

MOVi-A（または同等データ）を準備

ノートブック内の DATA_ROOT / DATASET_DIR などのパスを設定

baseline と disentangled で 同一 split / 前処理になっていることを確認

マスク系の指標（IoU）を評価する場合は、GT マスクが利用できるデータであることが望ましいです。
