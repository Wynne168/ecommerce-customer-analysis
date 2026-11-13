# ecommerce-customer-analysis

INT6163 高级数据分析小组项目。基于 Synerise 电商行为数据，分析客户行为趋势、客户分群和商品类目偏好，并用过去 28 天的行为预测客户未来 14 天是否购买。

## 项目目标

对预测时点前 28 天内至少有一次行为的活跃客户进行购买倾向排序。实验预设选择得分最高的 10% 客户，在相同选择人数下比较近期访问基线、近期性与频次（RF）规则、逻辑回归和直方图梯度提升树。预测实验按客户固定哈希抽样约 2%。

## 准备数据

从 [Synerise RecSys 2025 数据页面](https://recsys.synerise.com/data-set) 下载 **Synerise 原始数据集**，解压后将以下六个文件直接放入仓库根目录的 `data/raw/`。原始文件保存在本地。

| 文件 | 内容 |
| --- | --- |
| `page_visit.parquet` | 页面访问 |
| `search_query.parquet` | 搜索 |
| `add_to_cart.parquet` | 加入购物车 |
| `remove_from_cart.parquet` | 移出购物车 |
| `product_buy.parquet` | 购买行为 |
| `product_properties.parquet` | 商品属性 |

## 运行环境

项目记录的运行环境为 **Windows 11、Python 3.12.0**。依赖版本见 [requirements-verified.txt](requirements-verified.txt)，运行环境及验证状态见 [reports/environment.json](reports/environment.json)。

使用 Git 克隆仓库。在 Windows PowerShell 中运行以下命令，已有本地仓库时直接进入仓库根目录并从创建虚拟环境开始。

```powershell
git clone https://github.com/Wynne168/ecommerce-customer-analysis.git
cd ecommerce-customer-analysis
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-verified.txt
```

## Notebook 执行顺序

在安装了 Python 和 Jupyter 扩展的 VS Code 中打开仓库根目录和 [ecommerce_analysis.ipynb](notebooks/ecommerce_analysis.ipynb)，选择 `.venv` 对应的 Python 内核。重启内核后运行全部单元，按 **第 1 至 28 个代码格** 顺序执行。Notebook 会从当前目录向上查找 Git 仓库根目录。

| 代码格 | 内容 |
| --- | --- |
| 1–9 | 定位数据、检查数据质量、分析日度趋势与星期差异、核对商品关联 |
| 10–15 | 定义时间窗口、抽取客户样本、检查样本并构建特征与训练和验证标签 |
| 16–18 | 计算基线、训练逻辑回归和梯度提升树并比较验证集表现 |
| 19–22 | 绘制验证集曲线，检查同分边界、重复计数敏感性及 Bootstrap 差异 |
| 23–27 | 固定主模型后评价测试集，分析漏判、行为组重要性、客户分群和购买类目 |
| 28 | 导出汇总表和运行环境，检查图表文件 |

## 查看结果

- [figures/](figures/) 存放日度趋势、星期差异、验证集排序、测试集名单诊断、行为组重要性、客户分群和购买类目图表。
- [reports/tables/](reports/tables/) 存放 23 份汇总 CSV。重复计数敏感性见 [16_duplicate_sensitivity.csv](reports/tables/16_duplicate_sensitivity.csv)，最终测试比较见 [18_test_models.csv](reports/tables/18_test_models.csv)。
- `data/processed/` 存放运行生成的本地中间数据，客户级明细保留在本地。

在最终测试窗口中，逻辑回归从 117,048 名客户中选出排名前 10% 的 11,705 人，命中 816 位未来买家，覆盖全部 1,198 位未来买家的 68.11%。名单精确率为 6.97%，较 RF 规则多命中 245 位买家。

## 分析口径与限制

- 区分行为记录数、客户数与商品数。购买记录无法直接用于统计订单数，匿名价格档位也无法用于计算实际销售额。
- 页面访问 URL 缺少明确的商品映射，分析范围限于已记录的行为和可关联的商品事件。
- 特征仅使用预测时点之前的信息，测试结果用于评估固定模型。各方法在相同客户、标签和名单容量下比较。
- 10% 是实验设定的名单容量。实际触达效果、成本和收益需通过对照试点验证。
