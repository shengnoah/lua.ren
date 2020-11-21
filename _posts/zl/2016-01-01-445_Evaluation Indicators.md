---
layout: post
title: Evaluation Indicators 
tags: [lua文章]
categories: [topic]
---
假设测试集中有类别为A、B、C的三种class。现在要对class A 的recall、precision和AP进行评估。

$quad$ | class A | class not A  
---|---|---  
Test says “A” | True positive | False Positive |  
Test says “not A” | False positive | True Positive |  
  
公式为：  
`召回率 Recall = TP / (TP + FN)`  
`准确率 Precision = TP / (TP + FP)`

**也就是说，recall是一张图片中检测出来的目标占所有目标的比例，即“召回”的比例；precision是每个检测出来的目标的准确率。**

代码如下：  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    

|

    
    
    import numpy as np  
      
      
    def (preds, labels):  
        num_cls = preds.shape[1]  
        batch_size = preds.shape[0]  
      
        labels_hot = np.zeros((batch_size, num_cls))  
        labels_hot[np.arange(batch_size), labels] = 1  
      
        max_pred_idx = np.argmax(preds, axis=1)  
        preds_hot = np.zeros((batch_size, num_cls))  
        preds_hot[np.arange(batch_size), max_pred_idx] = 1  
      
        tp, tn, fp, fn = 0, 0, 0, 0  
        for i in range(num_cls):  
            pred = preds_hot[:, i]  
            label = labels_hot[:, i]  
            tp += int(np.sum(label[np.where(pred == label)[0]]))  
            tn += int(np.sum(1 - label[np.where(pred == label)[0]]))  
            fp += np.sum(label[np.where(pred == 1)] == 0)  
            fn += np.sum(label[np.where(pred == 0)] == 1)  
      
        precision = tp / (tp + fp)  
        recall = tp / (tp + fn)  
      
        return precision, recall  
      
  
---|---  
  
# 二、AP & mAP

单个类别的平均精度叫AP。先求出原precision-recall曲线的包络曲线，然后将转折点的precision值相加取平均即为该类的AP。  
而mAP就是将所有类别的AP相加取平均。

# 三、CMC

CMC（cumulated matching characteristic curve），即累计匹配特征曲线。

假如有一个类别为c1的样本，得到的匹配分数为：  
`c1:0.9 c2:0.8 c3:0.7`  
那么rank1（第一次即命中）=100%，则后面的rank2、rank3也为100%

如果得到的匹配分数为：  
`c1:0.8 c2:0.9 c3:0.7`  
那么rank1=0%，rank2=100%，rank3=100%

多类别的评估求和取平均即可。

**车辆reid的mAP和CMC代码如下：**  

    
    
     1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    54  
    55  
    56  
    57  
    58  
    59  
    60  
    61  
    62  
    63  
    64  
    65  
    66  
    67  
    68  
    69  
    70  
    71  
    72  
    73  
    74  
    75  
    76  
    

|

    
    
    def eval_market1501(distmat, q_pids, g_pids, q_camids, g_camids, max_rank):  
        """Evaluation with market1501 metric  
        Key: for each query identity, its gallery images from the same camera view are discarded.  
        """  
        num_q, num_g = distmat.shape  
        if num_g < max_rank:  
            max_rank = num_g  
            print("Note: number of gallery samples is quite small, got {}".format(num_g))  
        indices = np.argsort(distmat, axis=1)  
        matches = (g_pids[indices] == q_pids[:, np.newaxis]).astype(np.int32)  
      
          
        all_cmc = []  
        all_AP = []  
        num_valid_q = 0. # number of valid query  
        for q_idx in range(num_q):  
            # get query pid and camid  
            q_pid = q_pids[q_idx]  
            q_camid = q_camids[q_idx]  
      
            # remove gallery samples that have the same pid and camid with query  
            order = indices[q_idx]  
            remove = (g_pids[order] == q_pid) & (g_camids[order] == q_camid)  
            keep = np.invert(remove)  
      
            # compute cmc curve  
            orig_cmc = matches[q_idx][keep] # binary vector, positions with value 1 are correct matches  
            if not np.any(orig_cmc):  
                # this condition is true when query identity does not appear in gallery  
                continue  
      
            cmc = orig_cmc.cumsum()  
            cmc[cmc > 1] = 1  
      
            all_cmc.append(cmc[:max_rank])  
            num_valid_q += 1.  
      
            good_idx = []  
            keep_ap = (g_pids[order] == q_pid) & (g_camids[order] != q_camid)  
            for i in range(keep_ap.shape[0]):  
                if keep_ap[i]:  
                    good_idx.append(order[i])  
      
            # compute average precision  
            ngood = len(good_idx)  
            intersect_sz = 0  
            ap = 0  
            good_now = 0  
            old_recall = 0  
            old_precision = 0  
            counter = 0  
      
            for idx in order:  
                if idx in good_idx:  
                    intersect_sz += 1  
                    good_now += 1  
      
                recall = intersect_sz / ngood  
                precision = intersect_sz / (counter + 1)  
                ap += (recall - old_recall) * ((old_precision + precision) / 2)  
                old_recall = recall  
                old_precision = precision  
                counter += 1  
      
                if good_now == ngood:  
                    break  
      
            all_AP.append(ap)  
      
        assert num_valid_q > 0, "Error: all query identities do not appear in gallery"  
      
        all_cmc = np.asarray(all_cmc).astype(np.float32)  
        all_cmc = all_cmc.sum(0) / num_valid_q  
        mAP = np.mean(all_AP)  
      
        return all_cmc, mAP  
      
  
---|---