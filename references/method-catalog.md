# 单细胞与空间组学方法目录

## 使用说明

当用户点名一个分析模块时，只读取对应章节，并按“工具与计算 → 结果文件 → 可视化 → 统计边界”回答。R 与 Python 路线通常是替代关系，不要默认全部执行。函数名用于说明实现抓手，不代替正式代码与版本核验。

## 目录

1. 通用结果文件规范
2. 单细胞质控与污染处理
3. 标准化、降维与批次整合
4. 聚类、marker 与细胞注释
5. 细胞比例与差异丰度
6. 伪批量差异表达
7. 基因集打分与功能富集
8. 恶性细胞与CNV推断
9. 拟时序、RNA velocity与命运分析
10. 转录因子与调控网络
11. 细胞通讯
12. TCR/BCR免疫受体库
13. 空间转录组预处理与质控
14. 空间域与空间变异基因
15. 空间去卷积与单细胞映射
16. 空间邻域、共现与生态位
17. 空间细胞通讯
18. scATAC与多组学整合
19. bulk外部验证与去卷积
20. WGCNA、NMF与元程序
21. 生存、疗效预测与机器学习

## 1. 通用结果文件规范

- **对象文件**：R 保存 `*.rds` 或 `*.h5seurat`；Python 保存 `*.h5ad`；多组学可保存 `*.h5mu`；velocity 常保留 `*.loom` 或含 layers 的 `*.h5ad`。
- **表格文件**：统计结果保存为 UTF-8 `*.tsv`，至少包含 effect size、标准误/置信区间、原始 P 值和 FDR；矩阵过大时使用 `*.h5`、`*.mtx.gz` 或 `*.parquet`。
- **基因集/网络**：基因集保存 `*.gmt`；配体—受体、调控边和空间邻接保存 `*.tsv`；图结构可另存 `*.graphml`。
- **图形文件**：主图优先 `*.pdf`/`*.svg`，预览图使用 `*.png`；同时保留作图输入表。
- **元数据**：保存 `sample_metadata.tsv`、`cell_metadata.tsv` 或 `spot_metadata.tsv`，保留患者、组别、批次、区域与配对信息。
- **复现信息**：R 保存 `sessionInfo.txt`，Python 保存 `environment.yml` 或 `requirements.txt`，并记录随机种子和数据库版本。

推荐目录：`01_qc/`、`02_atlas/`、`03_composition/`、`04_state/`、`05_spatial/`、`06_communication/`、`07_clinical/`、`figures/`。

## 2. 单细胞质控与污染处理

**工具与计算**

- R 首选：`Seurat` + `scater/scuttle` 计算 nUMI、nGene、线粒体/核糖体比例；`DropletUtils::emptyDrops` 识别空液滴；`scDblFinder` 或 `DoubletFinder` 检测双细胞；`SoupX` 或 `celda::decontX` 校正环境RNA。
- Python 等价：`Scanpy` 计算QC指标与过滤；`Scrublet` 或 `scvi-tools::SOLO` 检测双细胞；`CellBender` 可用于原始液滴背景去除。
- 核心计算：按样本诊断低质量细胞、极端文库、双细胞和污染，不对所有组织机械使用同一阈值。

**结果文件**

- `object_raw.rds` / `adata_raw.h5ad`
- `qc_cell_metrics.tsv`
- `qc_sample_summary.tsv`
- `doublet_scores.tsv`
- `ambient_rna_correction.tsv`
- `object_qc.rds` / `adata_qc.h5ad`

**可视化**

- 每样本 nUMI、nGene、线粒体比例小提琴图/密度图。
- nUMI–nGene散点图、过滤前后细胞数柱状图。
- doublet score分布和UMAP叠加图。
- 污染校正前后典型基因表达对比热图。

**统计边界**：QC阈值属于技术决策，不应根据组别调到“结果更显著”；疾病组细胞质量系统性较差时应报告敏感性分析。

## 3. 标准化、降维与批次整合

**工具与计算**

- R首选：`Seurat` 的 `NormalizeData` 或 `SCTransform`，随后 PCA、`FindNeighbors`；跨样本可选 Seurat RPCA/CCA、`Harmony::RunHarmony` 或 `batchelor::fastMNN`。
- Python等价：`Scanpy` 的 `normalize_total`、`log1p`、HVG和PCA；批次整合可用 `harmonypy` 或 `scvi-tools::scVI`。
- 核心计算：保留原始counts用于差异分析；整合后的低维空间用于聚类和可视化，不直接把批次校正表达矩阵用于所有统计。

**结果文件**

- `hvg_genes.tsv`
- `pca_embeddings.tsv`
- `integration_embeddings.tsv`
- `object_integrated.rds` / `adata_integrated.h5ad`
- `integration_metrics.tsv`

**可视化**

- PCA/UMAP按样本、批次、组别和细胞类型着色。
- 校正前后样本混合度、LISI/kBET或邻域保持指标。
- 每个样本的细胞类型组成及整合后marker保留情况。

**统计边界**：组别与批次完全混杂时，任何算法都不能可靠恢复组别效应；避免过度整合抹去真实疾病状态。

## 4. 聚类、marker与细胞注释

**工具与计算**

- R首选：`Seurat` 的共享近邻图、Leiden/Louvain聚类和 `FindAllMarkers`；`SingleR` + `celldex` 或 Azimuth作参考映射，再结合人工marker审阅。
- Python等价：`Scanpy` 的邻居图、`tl.leiden`、`rank_genes_groups`；`CellTypist` 或 `scvi-tools::scANVI`进行参考注释。
- 核心计算：以多marker、参考映射、功能程序和组织知识共同注释；亚群命名记录证据及不确定性。

**结果文件**

- `cluster_assignments.tsv`
- `cluster_markers.tsv`
- `annotation_reference_scores.tsv`
- `celltype_annotation.tsv`
- `object_annotated.rds` / `adata_annotated.h5ad`

**可视化**

- UMAP/t-SNE细胞图谱。
- marker DotPlot、热图、FeaturePlot。
- 自动注释与人工注释混淆矩阵。
- 各样本/组别的细胞组成图。

**统计边界**：UMAP分离不等于新细胞类型；marker差异需排除批次、应激、细胞周期和低质量状态。

## 5. 细胞比例与差异丰度

**工具与计算**

- R首选：简单比例可用 `speckle::propeller`；邻域差异丰度用 `MiloR`；多条件/多细胞类型设计可用 `muscat`。
- Python等价：`pertpy` 中的 Milo/scCODA 工作流；组成数据可用 `scCODA`。
- 核心计算：以患者/动物为重复，比较细胞类型、亚群或邻域丰度；纳入配对、批次、年龄、性别等协变量。

**结果文件**

- `celltype_counts_by_sample.tsv`
- `celltype_proportions_by_sample.tsv`
- `da_celltype_group.tsv`
- `da_neighborhood_group.tsv`
- `da_model_diagnostics.tsv`

**可视化**

- 样本级箱线/小提琴叠加散点图。
- effect size与置信区间森林图。
- Milo邻域图和差异邻域蜂窝图。
- 组成热图、alluvial图作为描述性补充。

**统计边界**：不要把每个细胞当独立重复；组成数据中某类细胞上升会被动影响其他细胞比例。

## 6. 伪批量差异表达

**工具与计算**

- R首选：按“样本×细胞类型/亚群”汇总counts，使用 `edgeR`准似然、`DESeq2`或`limma-voom`；`muscat`可管理多样本多亚群设计。
- Python等价：用 `AnnData`按样本聚合counts，再用 `pydeseq2`；`Scanpy::rank_genes_groups`仅适合探索marker，不作为多患者主推断。
- 核心计算：为每个目标细胞类型建立设计矩阵，建模组别、配对、批次和临床协变量。

**结果文件**

- `pseudobulk_counts_celltype.tsv.gz`
- `pseudobulk_sample_metadata.tsv`
- `de_pseudobulk_celltype.tsv`
- `de_contrast_summary.tsv`

**可视化**

- 火山图、MA图、效应量热图。
- 关键基因样本级CPM/标准化表达箱线图。
- pseudobulk PCA/MDS图和样本相关性热图。

**统计边界**：细胞数过少的“样本×亚群”组合需过滤；报告logFC和FDR，不只展示单细胞小提琴图。

## 7. 基因集打分与功能富集

**工具与计算**

- R首选：单细胞打分用 `UCell`、`AUCell`或Seurat `AddModuleScore`；样本/伪批量通路用 `GSVA`；排序富集用 `fgsea`；ORA用 `clusterProfiler`，基因集可由 `msigdbr`获取；TF/通路活性可用 `decoupleR`。
- Python等价：`scanpy.tl.score_genes`、`decoupler`、`gseapy`。
- 核心计算：根据数据层级选择单细胞、样本级或排名式富集；记录基因集来源、物种和版本。

**结果文件**

- `genesets_used.gmt`
- `cell_signature_scores.tsv.gz`
- `sample_pathway_scores.tsv`
- `gsea_results.tsv`
- `ora_results.tsv`
- `leading_edge_genes.tsv`

**可视化**

- 通路dot plot、bar plot、NES热图。
- GSEA running-score曲线。
- signature在UMAP/空间切片上的FeaturePlot。
- 样本级通路评分箱线图及相关性热图。

**统计边界**：转录评分代表基因程序，不等于蛋白活性、代谢通量或功能结果；比较不同细胞类型时注意RNA含量差异。

## 8. 恶性细胞与CNV推断

**工具与计算**

- R首选：`infercnv`以正常参考细胞推断染色体大片段变化；`CopyKAT`区分非整倍体与二倍体细胞。
- Python等价：`infercnvpy`计算平滑CNV谱并聚类。
- 核心计算：联合上皮marker、病理来源、突变/CNV证据识别恶性细胞，不依赖单一CNV算法。

**结果文件**

- `cnv_cell_scores.tsv`
- `cnv_segment_matrix.tsv.gz`
- `malignant_cell_classification.tsv`
- `cnv_object.rds` / `adata_cnv.h5ad`

**可视化**

- 按染色体排列的CNV热图。
- CNV score在UMAP上的分布。
- 恶性/非恶性细胞比例及克隆树/亚克隆热图。

**统计边界**：转录组CNV是推断结果；低CNV肿瘤、正常增殖细胞和参考选择都可能影响分类。

## 9. 拟时序、RNA velocity与命运分析

**工具与计算**

- R首选：`Slingshot`推断谱系并用`tradeSeq`拟合沿轨迹基因；`Monocle3`学习主图；`destiny`可做diffusion map。
- Python首选：`scVelo`基于spliced/unspliced计数估计velocity；`Scanpy`的PAGA/DPT描述拓扑；`CellRank`估计起始、终末状态和命运概率。
- 核心计算：预先给出起点依据，比较轨迹算法或根节点敏感性；若无spliced/unspliced数据，不使用RNA velocity。

**结果文件**

- `trajectory_cell_pseudotime.tsv`
- `trajectory_lineages.tsv`
- `trajectory_gene_dynamics.tsv`
- `velocity_graph.h5ad`
- `cellrank_fate_probabilities.tsv`

**可视化**

- 轨迹UMAP、PAGA图、velocity stream/grid图。
- 基因/通路沿pseudotime平滑曲线。
- 分支热图、命运概率图、不同组细胞在轨迹上的密度图。

**统计边界**：拟时序和velocity提示状态方向，不等于真实谱系；需要真实时间、克隆、ATAC或谱系追踪增强证据。

## 10. 转录因子与调控网络

**工具与计算**

- R首选：`SCENIC`/`AUCell`推断regulon活性；`decoupleR`结合DoRothEA/VIPER资源估计TF活性。
- Python等价：`pySCENIC`（`arboreto` + motif pruning + AUCell）或`decoupler`。
- 核心计算：从TF—靶基因共表达和motif支持构建regulon，再比较细胞状态/组别的TF活性。

**结果文件**

- `regulon_definitions.tsv`
- `regulon_auc_matrix.tsv.gz`
- `tf_activity_by_cell.tsv.gz`
- `differential_tf_activity.tsv`
- `regulatory_network.graphml`

**可视化**

- TF活性热图、UMAP FeaturePlot。
- regulon specificity score排名。
- TF—靶基因网络图、关键靶基因表达图。

**统计边界**：共表达与motif支持不等于直接调控；关键TF需用ATAC、ChIP或扰动实验验证。

## 11. 细胞通讯

**工具与计算**

- R首选：`CellChat`分析全局通路和发送/接收角色；`NicheNet`分析配体到接收细胞靶基因；多样本条件比较优先`MultiNicheNet`。
- Python等价：`LIANA+`整合多个LR方法；`CellPhoneDB`做置换检验和特异互作；空间场景可与`COMMOT`联用。
- 核心计算：先限定发送者、接收者和主比较，再结合配体、受体、下游靶基因及空间邻近筛选候选轴。

**结果文件**

- `lr_interactions_all.tsv`
- `lr_differential_group.tsv`
- `sender_receiver_scores.tsv`
- `ligand_target_matrix.tsv`
- `communication_pathway_summary.tsv`

**可视化**

- 聚焦的sender–receiver气泡图或热图。
- 配体活性排名、ligand–target热图。
- 通路级circle/chord图作为补充。
- 关键LR在不同细胞中的DotPlot及空间共定位图。

**统计边界**：LR表达只能提出通讯候选；避免把全网络“毛线球”作为核心证据，复杂受体需检查亚基完整性。

## 12. TCR/BCR免疫受体库

**工具与计算**

- R首选：`scRepertoire`整合Seurat并分析克隆扩增、共享和多样性；`immunarch`用于群体级免疫组库统计；BCR谱系可结合`alakazam/Change-O`。
- Python等价：`scirpy`结合AnnData分析克隆型、克隆扩增、V/J使用和细胞状态。
- 核心计算：定义克隆规则，比较组织/组别/亚群间扩增、共享、迁移与状态转换。

**结果文件**

- `clonotype_assignments.tsv`
- `clonal_expansion_by_sample.tsv`
- `clonotype_overlap_matrix.tsv`
- `vdj_usage.tsv`
- `bcr_shm_isotype.tsv`

**可视化**

- clonotype size堆叠图、克隆扩增UMAP。
- 克隆共享Circos/alluvial图。
- Morisita/Jaccard重叠热图、V/J使用图和多样性箱线图。

**统计边界**：克隆共享提示共同来源或迁移，但不能单独确定方向；跨患者通常不能直接比较具体CDR3克隆。

## 13. 空间转录组预处理与质控

**工具与计算**

- R首选：`Seurat`或`SpatialExperiment`导入表达、坐标和图像；`scater`计算spot QC；高分辨率图像可结合`Giotto`。
- Python等价：`Scanpy` + `Squidpy`读取Visium/空间对象、计算QC和图像特征；`stLearn`可做形态与表达联合处理。
- 核心计算：评估每张切片的reads、UMI、基因数、组织覆盖、背景spot、线粒体比例和空间异常区域。

**结果文件**

- `spatial_object_qc.rds` / `spatial_qc.h5ad`
- `spot_qc_metrics.tsv`
- `slide_qc_summary.tsv`
- `tissue_positions.tsv`
- `pathology_roi_annotations.tsv`

**可视化**

- H&E叠加UMI、基因数和线粒体比例空间图。
- 每张切片QC分布、过滤前后spot图。
- 组织区域、ROI和边界标注图。

**统计边界**：spot嵌套在切片和患者内；跨组比较不能把所有spot视为独立患者。

## 14. 空间域与空间变异基因

**工具与计算**

- R首选：`BayesSpace`提高spot级空间聚类分辨率；`BANKSY`或`Giotto`结合表达与邻域识别空间域；空间变异基因用`SPARK-X`或`nnSVG`。
- Python等价：`SpaGCN`、`STAGATE`或`stLearn`识别空间域；`Squidpy`计算Moran's I；`SpatialDE`识别空间变异基因。
- 核心计算：构建空间邻接图，识别重复出现的组织域，并比较域内表达、通路和组间丰度。

**结果文件**

- `spatial_domain_assignments.tsv`
- `spatial_domain_markers.tsv`
- `spatially_variable_genes.tsv`
- `spatial_domain_abundance_by_sample.tsv`
- `spatial_embeddings.tsv`

**可视化**

- H&E叠加空间域图。
- 空间域marker热图、SVG空间FeaturePlot。
- 不同患者/组别的域比例箱线图。
- 域间邻接图和空间嵌入图。

**统计边界**：空间域需要病理和marker解释；算法生成的cluster编号本身没有生物学意义。

## 15. 空间去卷积与单细胞映射

**工具与计算**

- R首选：`spacexr::RCTD`估计spot细胞组成；`SPOTlight`用NMF回归；`CARD`利用空间相关性平滑丰度。
- Python首选：`cell2location`进行贝叶斯细胞丰度估计；`Tangram`映射单细胞到空间；`scvi-tools::DestVI`解析连续细胞状态。
- 核心计算：使用匹配组织的高质量scRNA参考，检查共线细胞类型、参考缺失和每spot细胞数先验。

**结果文件**

- `deconvolution_celltype_abundance.tsv.gz`
- `deconvolution_uncertainty.tsv.gz`
- `cell_to_space_mapping.tsv.gz`
- `reference_signature_matrix.tsv`
- `spatial_object_deconvolved.rds` / `spatial_deconvolved.h5ad`

**可视化**

- 每种细胞类型的空间丰度图。
- spot组成堆叠饼图/条形图用于局部展示。
- 预测丰度与marker表达相关性图。
- 多方法或原位验证的一致性热图。

**统计边界**：去卷积结果是模型估计；相近转录状态可能不可辨，必须报告不确定性并用marker/成像验证。

## 16. 空间邻域、共现与生态位

**工具与计算**

- R首选：`MISTyR`分析同spot、近邻和远距离多视图依赖；`Giotto`做细胞邻近富集；高分辨率成像可用`lisaClust/spicyR`识别细胞邻域并比较组别。
- Python首选：`Squidpy`构建空间图并计算neighborhood enrichment、co-occurrence、Ripley's statistics和中心性。
- 核心计算：预定义邻域半径/邻居数，使用保持组织结构的置换null model，识别跨患者可复现的共现、排斥和生态位。

**结果文件**

- `spatial_neighbor_graph.tsv.gz`
- `neighborhood_enrichment.tsv`
- `celltype_cooccurrence.tsv`
- `spatial_niche_assignments.tsv`
- `spatial_niche_abundance.tsv`
- `distance_to_structure.tsv`

**可视化**

- 邻域富集热图、共现曲线。
- 细胞/spot生态位空间图。
- 最近距离分布、核心—边缘梯度曲线。
- niche组成热图及样本级丰度森林图。

**统计边界**：邻近可能由共同偏好同一解剖结构造成；需控制区域、细胞密度和边界效应。

## 17. 空间细胞通讯

**工具与计算**

- R路线：先用`CellChat`/`MultiNicheNet`得到条件特异LR，再用空间邻接、距离或`MISTyR`限制到真实邻域。
- Python首选：`COMMOT`基于最优传输估计空间通讯方向与强度；`LIANA+`可整合LR结果并加入空间先验；`stLearn`也支持空间LR分析。
- 核心计算：联合发送者配体、接收者受体、邻接关系和接收者下游响应，比较疾病组与对照组。

**结果文件**

- `spatial_lr_pairs.tsv`
- `spatial_lr_scores_by_spot.tsv.gz`
- `communication_direction_vectors.tsv`
- `spatial_ligand_target_results.tsv`
- `spatial_lr_group_comparison.tsv`

**可视化**

- 关键LR空间共表达图和局部放大图。
- 通讯方向vector/stream图。
- sender–receiver–target Sankey图或聚焦网络图。
- 组间LR强度箱线图及跨患者森林图。

**统计边界**：同一spot共表达不等于直接通讯；核心轴仍需原位蛋白/转录本和干预实验验证。

## 18. scATAC与多组学整合

**工具与计算**

- R首选：`Signac`进行TF-IDF/LSI、peak calling接口、gene activity和WNN；`ArchR`完成质控、聚类、peak-to-gene link和轨迹；`chromVAR`计算motif活性。
- Python等价：`muon`处理MuData和WNN；`SnapATAC2`做大规模scATAC；`scvi-tools::MultiVI`联合RNA与ATAC。
- 核心计算：识别开放染色质、motif活性、peak—gene联系和细胞状态特异调控元件。

**结果文件**

- `atac_object.rds` / `multiome.h5mu`
- `peak_matrix.mtx.gz`
- `peaks.bed`
- `motif_activity.tsv.gz`
- `peak_gene_links.tsv`
- `wnn_embeddings.tsv`

**可视化**

- scATAC UMAP、fragment/TSS QC图。
- motif活性热图/FeaturePlot。
- 基因位点coverage plot、peak—gene弧线图。
- RNA与ATAC联合UMAP和调控网络图。

**统计边界**：gene activity和peak—gene link是推断；关键调控关系需与表达、motif及实验证据一致。

## 19. bulk外部验证与去卷积

**工具与计算**

- R首选：`DESeq2/edgeR/limma`处理bulk差异；`GSVA`计算单细胞signature；`MuSiC`或`BisqueRNA`用scRNA参考去卷积；免疫去卷积可用`immunedeconv`统一接口。
- Python等价：`pydeseq2`做差异，`gseapy/decoupler`做signature评分；复杂去卷积可调用对应独立工具并统一整理结果。
- 核心计算：冻结单细胞发现的marker/signature，在独立bulk队列评估细胞状态、临床表型和疗效关联。

**结果文件**

- `signature_genes_frozen.gmt`
- `bulk_signature_scores.tsv`
- `bulk_deconvolution_fractions.tsv`
- `bulk_de_results.tsv`
- `external_validation_statistics.tsv`

**可视化**

- signature评分箱线图、队列间森林图。
- 去卷积细胞比例热图/箱线图。
- signature与临床指标相关性图、KM曲线或疗效瀑布图。

**统计边界**：外部验证前固定signature和方向；同一数据集不能既筛选signature又被称为独立验证。

## 20. WGCNA、NMF与元程序

**工具与计算**

- R首选：样本级网络用`WGCNA`；单细胞metacell网络用`hdWGCNA`；表达程序分解可用`NMF`或`RcppML`。
- Python等价：`cNMF`识别稳定表达程序；`sklearn.decomposition.NMF`用于可控的矩阵分解。
- 核心计算：识别共表达模块或元程序，计算模块得分、hub基因及其与表型/空间生态位的关联。

**结果文件**

- `module_assignments.tsv`
- `module_eigengenes.tsv`
- `module_trait_correlations.tsv`
- `hub_genes.tsv`
- `nmf_program_genes.tsv`
- `nmf_program_usage.tsv`

**可视化**

- 模块—表型相关性热图。
- 模块基因网络图、hub gene排名。
- NMF程序基因热图、program usage UMAP/空间图。

**统计边界**：网络和元程序依赖样本数、稀疏度与分解秩；hub不自动等于因果驱动基因。

## 21. 生存、疗效预测与机器学习

**工具与计算**

- R首选：`survival`拟合Cox，`survminer`绘KM；`glmnet`做惩罚回归；`timeROC`评估时间依赖AUC；`rms/riskRegression`做校准和临床模型评估；机器学习可用`tidymodels`或`mlr3`。
- Python等价：`lifelines`、`scikit-survival`；分类/回归用`scikit-learn`、`xgboost`，解释用`shap`。
- 核心计算：预定义特征，进行训练/验证划分或嵌套交叉验证，比较加入组学特征前后的临床模型。

**结果文件**

- `clinical_model_features.tsv`
- `cox_results.tsv`
- `model_predictions.tsv`
- `cross_validation_metrics.tsv`
- `calibration_results.tsv`
- `external_validation_metrics.tsv`
- `shap_values.tsv`

**可视化**

- KM曲线、Cox森林图。
- ROC/timeROC、PR曲线、校准曲线、决策曲线。
- 特征重要性/SHAP图、患者风险排序图。

**统计边界**：特征选择必须在训练折内完成；报告判别、校准和外部验证，不用同一队列同时定阈值和验证。

## 输出示例

用户问“如何做细胞通讯分析”时，回答应类似：

1. **首选工具**：多样本组间比较用R包`MultiNicheNet`，全局网络补充用`CellChat`；Python等价可用`LIANA+`。
2. **计算**：指定发送者与接收者，比较疾病和对照的配体—受体活性，并用NicheNet式ligand-to-target分析检查接收细胞下游响应。
3. **结果文件**：`lr_differential_group.tsv`、`ligand_target_matrix.tsv`、`sender_receiver_scores.tsv`和带注释的单细胞对象。
4. **可视化**：差异LR气泡图、ligand–target热图、关键通路sender–receiver图、关键基因DotPlot；有空间数据时增加LR共定位图。
5. **边界**：这些结果提出候选通讯轴，不单独证明直接作用。
