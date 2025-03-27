# Notes on TreeLUT

ACM FPGA25’

## Part 1: Motivation and Objectives

## 1.1 Motivation

*   Machine learning is computation intensive, needs device like FPGA to accelerate
*   Former works are done to implement NNs on FPGA. These approaches need heavily quantized (thus hurting accuracy) and have time-consuming training process
*   GBDT performs equally well (outperform in some cases) as NNs
*   Former works of GBDT’s implementation on FPGA are either complex memory-based or have dramatic synthesis time

## 1.2 Objectives

*   Open source python library: training model -> RTL (edge to edge)
*   3-layer architecture: modular, scalable and efficient
*   Insert pipelining stages

## Part 2: Innovation and Contribution

## 2.1 Pre-knowledge

1.  Grandient Boosting Decision Trees (GBDT)

    *   DT![\<img alt="" data-attachment-key="L8WLKSAT" width="708" height="609" src="attachments/L8WLKSAT.png" ztype="zimage"> | 708](attachments/L8WLKSAT.png)

    *   GB![\<img alt="" data-attachment-key="72LZCXJ3" width="716" height="680" src="attachments/72LZCXJ3.png" ztype="zimage"> | 716](attachments/72LZCXJ3.png)the final prediction value is given as follows:

        $$
        F(X) = f_0 + \sum^{M}_{m=1}f_m(X)
        $$

    *   Binary classification = regression + sigmoid

    *   Multi-class classification = one-vs-all + softmax

2.  Quantization

    *   Threshold Quantization

        *   Boosting algorithm gives quantized threshold if the training data is quantized

        *   Quantization before training

        *   min-max normalization + w\_feature quantization:

            $$
            X_{normalization} = \frac{X-min(X)}{max(X) - min(X)}
            $$



            $$
            X_{quantizied} = round(X_{normalization}\times (2^{w\_feature}-1))
            $$

    *   Binary Classification Leaf Quantization

        *   Use the sum of every tree’s minimum leaf value for shifting

            $$
            \forall m \in {1,...,M},\ minLeaf_m = min_{X}(f_m(X))
            $$



            $$
            \begin{aligned}
            F(X) = & f_0 + \sum_{m = 1}f_m(X) & \\
            	 = & f_0 + \sum_{m=1}^{M}[f_m(X) - minLeaf_m] + \sum_{m=1}^{M}minLeaf_m & \\
            = & [f_0 + \sum_{m=1}^{M}minLeaf_m] + \sum_{m=1}^{M}[f_m(X) - minLeaf_m] & \\
            F(X) = & b + \sum_{m = 1}f_m'(X) & \\
            \end{aligned}

            $$

        *   Use the maximum of all tree’s biggest leaf value for scalingHyper parameter: w\_tree

            $$
            binaryScale = \frac{2^{w\_tree}-1}{max_{i,X}(f_i'(X))}
            $$



            $$
            \begin{aligned}F^{\prime}(X) & =F(X) \times \text { binaryScale } \\& =b \times \text { binaryScale }+\sum_{m=1}^{M} f_{m}^{\prime} \times \text { binaryScale } \\F^{\prime}(X) & =b^{\prime}+\sum_{m=1}^{M} f_{m}^{\prime \prime}(X)\end{aligned}
            $$



            $$
            \begin{array}{l}
            F^{\prime}(X) \approx Q F(X)=\operatorname{round}\left(b^{\prime}\right)+\sum_{m=1}^{M} \operatorname{round}\left(f_{m}^{\prime \prime}(X)\right) \\
            Q F(X)=q b+\sum_{m=1}^{M} q f_{m}(X)
            \end{array}
            $$

        *   example:

            $$
            \hat{y} \approx\left\{\begin{array}{ll}
            1 & Q F(X) \geq 0 \\
            0 & Q F(X)<0
            \end{array} \quad ; Q F(X)=q b+\sum_{m=1}^{M} q f_{m}(X)\right.
            $$

            ![\<img alt="" data-attachment-key="UXX9PQKL" width="624" height="246" src="attachments/UXX9PQKL.png" ztype="zimage"> | 624](attachments/UXX9PQKL.png)

    *   Multi-class Classification Leaf Quantization

        *   the classifier is different:

            $$
            \hat{y}=\operatorname{argmax}_{n}\left(F_{n}(X)\right) ; F_{n}(X)=f_{0}+\sum_{m=1}^{M} f_{n, m}(X)
            $$

        *   Use the sum of every tree’s minimum leaf value for shifting (for each class)

            $$
            \begin{aligned}
            F(X) = & f_0 + \sum_{m = 1}f_{n,m}(X) & \\
            	 = & f_0 + \sum_{m=1}^{M}[f_{n,m}(X) - minLeaf_{n,m}] + \sum_{m=1}^{M}minLeaf_{n,m} & \\
            = & [f_0 + \sum_{m=1}^{M}minLeaf_{n,m}] + \sum_{m=1}^{M}[f_{n,m}(X) - minLeaf_{n,m}] & \\
            F(X) = & b + \sum_{m = 1}f_{n,m}'(X) & \\
            \end{aligned}

            $$

        *   Use the maximum of all tree’s biggest leaf value for scaling (for all classes)Hyper parameter: w\_tree

            $$
            multiScale = \frac{2^{w\_tree}-1}{max_{i,j,X}(f_{i,j}'(X))}
            $$



            $$
            \begin{aligned}F_n^{\prime}(X) & =F_n(X) \times \text { multiScale } \\& =b_n \times \text { multiScale }+\sum_{m=1}^{M} f_{n,m}^{\prime} \times \text { multiScale } \\F_n^{\prime}(X) & =b_n^{\prime}+\sum_{m=1}^{M} f_{n,m}^{\prime \prime}(X)\end{aligned}
            $$

        *   final result:

            $$
            \begin{array}{l}
            F_n^{\prime}(X) \approx Q F_n(X)=\operatorname{round}\left(b_n^{\prime}\right)+\sum_{m=1}^{M} \operatorname{round}\left(f_{n,m}^{\prime \prime}(X)\right) \\
            Q F_n(X)=q b_n+\sum_{m=1}^{M} q f_{n,m}(X)
            \end{array}
            $$



            $$
            \hat{y}=\operatorname{argmax}_{n}\left(QF_{n}(X)\right) ; QF_{n}(X)=qb_n+\sum_{m=1}^{M} qf_{n, m}(X)
            $$

3.  Hardware Architecture

    *   Overall![\<img alt="" data-attachment-key="UTY9G3F4" width="632" height="1191" src="attachments/UTY9G3F4.png" ztype="zimage"> | 632](attachments/UTY9G3F4.png)

    *   Key Generator

        *   fully unrolled parallel comparators, each comparator’s output is 1 bit![\<img alt="" data-attachment-key="KEHVTXCB" width="633" height="266" src="attachments/KEHVTXCB.png" ztype="zimage"> | 633](attachments/KEHVTXCB.png)

    *   Decision Tree

        *   fully unrolled parallel sets of MUX, each tree’s input is the key generated before:![\<img alt="" data-attachment-key="JGE7TDJ9" width="635" height="360" src="attachments/JGE7TDJ9.png" ztype="zimage"> | 635](attachments/JGE7TDJ9.png)

    *   Adder

        *   Use adder tree structure
        *   In binary classification, we can compare bias with accumulated trees’ output instead of adding up bias

4.  Pipelining

    *   \[p0, p1 ,p2] are hyper-parameters

    *   <span class="highlight" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2FI6CVLCM6%22%2C%22pageLabel%22%3A%225%22%2C%22position%22%3A%7B%22pageIndex%22%3A5%2C%22rects%22%3A%5B%5B419.573%2C304.4404609%2C558.1978478719998%2C313.81924480000004%5D%2C%5B317.955%2C294.4318288%2C548.4699539328001%2C302.37605920000004%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2F3SWQS9A4%22%5D%2C%22locator%22%3A%225%22%7D%7D" ztype="zhighlight"><a href="zotero://open/library/items/I6CVLCM6?page=6">“p0 and p1 show if there are any pipelining stages after the key generator and decision tree modules, respectively.”</a></span><span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2F3SWQS9A4%22%5D%2C%22locator%22%3A%225%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/3SWQS9A4">Khataei 和 Bazargan, 2025, p. 5</a></span>)</span>

    *   The parameter p2 controls the number of pipelining stages in adder tree modules. Rather than inserting registers at every level, they are uniformly distributed across select levels according to the adder tree depth. For instance, when the adder tree depth is 6 and p2=1, only the third level contains a pipelining stage after the adders.

2.2 Tool Flow

A python library(edge to edge) is open-sourced. Users first decide quantization parameters and boosting parameters to obtain a target accuracy. Then, decide the pipeline parameters manually.

![\<img alt="" data-attachment-key="QIPPNRMV" width="620" height="619" src="attachments/QIPPNRMV.png" ztype="zimage"> | 620](attachments/QIPPNRMV.png)

## Part 3: Performance

3.1 Model Training and Quantization

![\<img alt="" data-attachment-key="PAGBAEXX" width="1279" height="367" src="attachments/PAGBAEXX.png" ztype="zimage"> | 1279](attachments/PAGBAEXX.png)

![\<img alt="" data-attachment-key="6LNAVCX6" width="629" height="639" src="attachments/6LNAVCX6.png" ztype="zimage"> | 629](attachments/6LNAVCX6.png)

3.2 Hardware Implementation

*   <span class="highlight" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2FI6CVLCM6%22%2C%22pageLabel%22%3A%226%22%2C%22position%22%3A%7B%22pageIndex%22%3A6%2C%22rects%22%3A%5B%5B512.373718016%2C234.3688288%2C558.2029610240002%2C242.3130592%5D%2C%5B317.74%2C223.40982879999999%2C558.1982252799999%2C231.3540592%5D%2C%5B317.955%2C212.45082879999998%2C517.4532037248%2C220.3950592%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2F3SWQS9A4%22%5D%2C%22locator%22%3A%226%22%7D%7D" ztype="zhighlight"><a href="zotero://open/library/items/I6CVLCM6?page=7">“we used the xcvu9p-flgb2104-2-i FPGA part with the Flow_PerfOptimized_high settings in the Out-of-Context (OOC) synthesis mode.”</a></span><span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2F3SWQS9A4%22%5D%2C%22locator%22%3A%226%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/3SWQS9A4">Khataei 和 Bazargan, 2025, p. 6</a></span>)</span>

3.3 Discussion

![\<img alt="" data-attachment-key="V32RAHJK" width="1297" height="1108" src="attachments/V32RAHJK.png" ztype="zimage"> | 1297](attachments/V32RAHJK.png)

![\<img alt="" data-attachment-key="V4G3DCTX" width="619" height="1205" src="attachments/V4G3DCTX.png" ztype="zimage"> | 619](attachments/V4G3DCTX.png)

TreeLUT performs better in terms of scalability:

*   Hardware cost hardly increases as accuracy increase
*   TreeLUT tool directly generate RTL without HLS
*   A simpler version of GBDT is used here

TreeLUT has lower area-delay product then DWN when they have the same accuracy.

<span class="highlight" data-annotation="%7B%22attachmentURI%22%3A%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2FI6CVLCM6%22%2C%22pageLabel%22%3A%229%22%2C%22position%22%3A%7B%22pageIndex%22%3A9%2C%22rects%22%3A%5B%5B327.918%2C596.0128287999999%2C559.7126140800001%2C603.9570591999999%5D%2C%5B317.955%2C585.0538288%2C558.2041288319996%2C592.9980592000001%5D%2C%5B317.955%2C574.0948288%2C545.9331204480001%2C582.0390592%5D%5D%7D%2C%22citationItem%22%3A%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2F3SWQS9A4%22%5D%2C%22locator%22%3A%229%22%7D%7D" ztype="zhighlight"><a href="zotero://open/library/items/I6CVLCM6?page=10">“DWN [8] is a recent LUT-based NN work. In this method, input data is binarized using a distributive thermometer encoding scheme [7], consisting of a set of comparisons with thresholds.”</a></span> <span class="citation" data-citation="%7B%22citationItems%22%3A%5B%7B%22uris%22%3A%5B%22http%3A%2F%2Fzotero.org%2Fusers%2F16695520%2Fitems%2F3SWQS9A4%22%5D%2C%22locator%22%3A%229%22%7D%5D%2C%22properties%22%3A%7B%7D%7D" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/3SWQS9A4">Khataei 和 Bazargan, 2025, p. 9</a></span>)</span>
