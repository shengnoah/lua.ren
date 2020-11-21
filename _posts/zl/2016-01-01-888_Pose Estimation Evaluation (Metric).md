---
layout: post
title: Pose Estimation Evaluation (Metric) 
tags: [lua文章]
categories: [topic]
---
对于人体骨骼关键点算法来说，如何有效的衡量一个算法的好坏非常重要，他不像分类问题那样可以很容易采用一些常用指标，例如precise、error、F-score等进行计算。因为在衡量的过程中，我们无法有效的将预测结果与真实值一一对应，无法知道他们之前的对应关系，更加无法知道当前的某个预测结果是否出现了误检或者漏检的情况。因此，构建一个合适的人体关键点的度量指标很重要。

本文目前涉及 **oks、AP、mAP、PCK、PCKh、PDJ** ，后续碰到新的会继续补充

* * *

#### 一、oks

目前最为常用的就是 **OKS（Object Keypoint Similarity）**
指标，这个指标启发于目标检测中的IoU指标，目的就是为了计算真值和预测人体关键点的相似度。

#### OKS

![\[公式\]](https://www.zhihu.com/equation?tex=OKS_%7Bp%7D+%3D+%5Cfrac%7B%5Csum_i+exp%5C%7B-d_%7Bpi%7D%5E2+%2F+2S_p%5E2+%5Csigma_i%5E2%5C%7D%5Cdelta%28v_%7Bpi%7D%3D1%29%7D%7B%5Csum_i+%5Cdelta%28v_%7Bpi%7D%3D1%29%7D)

其中：

![\[公式\]](https://www.zhihu.com/equation?tex=p) 表示groudtruth中，人的id

![\[公式\]](https://www.zhihu.com/equation?tex=i) 表示key point的id

![\[公式\]](https://www.zhihu.com/equation?tex=d_%7Bpi%7D)
表示groudtruth中每个人和预测的每个人的关键点的欧氏距离

![\[公式\]](https://www.zhihu.com/equation?tex=S_p)
表示当前人的尺度因子，这个值等于此人在groundtruth中所占面积的平方根，即
![\[公式\]](https://www.zhihu.com/equation?tex=%5Csqrt%7B%28x_2-x_1%29%28y_2-y_1%29%7D)

![\[公式\]](https://www.zhihu.com/equation?tex=%5Csigma_i) 表示第 i
个骨骼点的归一化因子，这个因此是通过对数据集中所有groundtruth计算的标准差而得到的，反映出当前骨骼点标注时候的标准差，
![\[公式\]](https://www.zhihu.com/equation?tex=%5Csigma) 越大表示这个点越难标注。

![\[公式\]](https://www.zhihu.com/equation?tex=v_%7Bpi%7D)代表第p个人的第 i 个关键点是否可见

![\[公式\]](https://www.zhihu.com/equation?tex=%5Cdelta) 用于将可见点选出来进行计算的函数

#### OKS矩阵

上面介绍的OKS是计算两个人之间的骨骼点相似度的，那一张图片中有很多的人时，该怎么计算呢？这时候就是构造一个OKS矩阵了。

假设一张图中，一共有M个人（groudtruth中），现在算法预测出了N个人，那么我们就构造一个M×N的矩阵，矩阵中的位置（i,j）代表groudtruth中的第i个人和算法预测出的第j个人的OKS相似度，找到矩阵中每一行的最大值，作为对应于第i个人的OKS相似度值。

#### AP（Average Precision）

根据前面的OKS矩阵，已经知道了某一张图像的所有人（groundtruth中出现的）的OKS分数，现在测试集中有很多图像，每张图像又有一些人，此时该如何衡量整个算法的好坏的。这个时候就用到了AP的概念，AP就是给定一个t，如果当前的OKS大于t，那就说明当前这个人的骨骼点成功检测出来了，并且检测对了，如果小于t，则说明检测失败或者误检漏检等，因此对于所有的OKS，统计其中大于t的个数，并计算其占所有OKS的比值。即假设OKS一共有100个，其中大于阈值t的共有30个，那么AP值就是30/100=0.3.

#### mAP（mean Average Precision）

顾名思义，AP的均值，具体计算方法就是给定不同的阈值t，计算不同阈值情况下对应的AP，然后求个均值就ok了。（ **与分类的 mAP 指标不同** ）

* * *

### 二、PCK,PCKh，PDJ

`MPII 中是以头部长度(head length) 作为归一化参考，即 PCKh.`

`FLIC 中是以躯干直径(torso size) 作为归一化参考.`

#### **1.PCK (Percentage of Correct Keypoints)**

关键点正确估计的比例  
计算检测的关键点与其对应的groundtruth间的归一化距离小于设定阈值的比例(the percentage of detections that
fall within a normalized distance of the ground truth).

    
    
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
    

|

    
    
    function eval_pck(pred, joints, symmetry_joint_id, joint_name, name)  
    % PCK 的实现  
    % torso height: || left_shoulder - right hip ||  
    % symmetry_joint_id: 具有对称关系的关键点 ID  
    % joint_name: 具有对称关系的关键点名字  
      
    range = 0:0.01:0.1;  
    show_joint_ids = (symmetry_joint_id >= 1:numel(symmetry_joint_id));  
      
    % compute distance to ground truth joints  
    dist = get_dist_pck(pred, joints(1:2,:,:));  
      
    % 计算 PCK  
    pck_all = compute_pck(dist,range);  
    pck = pck_all(end, :);  
    pck(1:end-1) = (pck(1:end-1) + pck(symmetry_joint_id))/2;  
      
    % 可视化结果  
    pck = [pck(show_joint_ids) pck(end)];  
    fprintf('------------ PCK Evaluation: %s -------------n', name);  
    fprintf('Parts '); fprintf('& %s ', joint_name{:}); fprintf('& Meann');  
    fprintf('PCK   '); fprintf('& %.1f  ', pck); fprintf('n');  
      
    % -------------------------------------------------------------------------  
    function dist = get_dist_pck(pred, gt)  
    assert(size(pred,1) == size(gt,1) && size(pred,2) == size(gt,2) && size(pred,3) == size(gt,3));  
      
    dist = nan(1,size(pred, 2), size(pred,3));  
      
    for imgidx = 1:size(pred,3)  
      % torso diameter 躯干直径  
      if size(gt, 2) == 14  
        refDist = norm(gt(:,10,imgidx) - gt(:,3,imgidx));  
      elseif size(gt, 2) == 10 % 10 joints FLIC  
        refDist = norm(gt(:,7,imgidx) - gt(:,6,imgidx));  
      elseif size(gt, 2) == 11 % 11 joints FLIC  
        refDist = norm(gt(:,4,imgidx) - gt(:,11,imgidx));  
      else  
        error('Number of joints should be 14 or 10 or 11');  
      end  
      
      % 预测的关键点与 gt 关键点的距离  
      dist(1,:,imgidx) = sqrt(sum((pred(:,:,imgidx) - gt(:,:,imgidx)).^2,1))./refDist;  
      
    end  
      
    % -------------------------------------------------------------------------  
    function pck = compute_pck(dist,range)  
    pck = zeros(numel(range),size(dist,2)+1);  
      
    for jidx = 1:size(dist,2)  
      % 计算每个设定阈值的 PCK  
      for k = 1:numel(range)  
        pck(k,jidx) = 100*mean(squeeze(dist(1,jidx,:)) <= range(k));  
      end  
    end  
      
    % 计算平均 PCK  
    for k = 1:numel(range)  
      pck(k,end) = 100*mean(reshape(squeeze(dist(1,:,:)),size(dist,2)*size(dist,3),1) <= range(k));  
    end  
      
  
---|---  
  
#### 1.1 PCKh

    
    
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
    

|

    
    
    % 计算 head 的长度  
    headSize = getHeadSizeAll(annolist_test_flat(single_person_test_flat == 1));  
      
    % 计算预测的关键点与 gt 关键点间的归一化距离  
    dist = getDistPCKh(pred,gt,headSize);  
      
    % 计算 PCKh  
    pck = computePCK(dist,range);  
      
      
    function headSizeAll = getHeadSizeAll(annolist)  
    headSizeAll = nan(length(annolist),1);  
    for imgidx = 1:length(annolist)  
        rect = annolist(imgidx).annorect;  
        headSizeAll(imgidx) =  util_get_head_size(rect);  
    end  
    end  
      
    function headSize = util_get_head_size(rect)  
    SC_BIAS = 0.6; % 0.8*0.75  
    headSize = SC_BIAS*norm([rect.x2 rect.y2] - [rect.x1 rect.y1]);   
    end  
      
    function dist = getDistPCKh(pred,gt,refDist)  
    assert(size(pred,1) == size(gt,1) && size(pred,2) == size(gt,2) && size(pred,3) == size(gt,3));  
    assert(size(refDist,1) == size(gt,3));  
      
    dist = nan(1,size(pred,2),size(pred,3));  
    for imgidx = 1:size(pred,3)  
        % pred joints 与 gt joints 的归一化距离  
        dist(1,:,imgidx) = sqrt(sum((pred(:,:,imgidx) - gt(:,:,imgidx)).^2,1))./refDist(imgidx);  
    end  
      
    function pck = computePCK(dist,range)  
    pck = zeros(numel(range),size(dist,2)+2);  
    for jidx = 1:size(dist,2)  
        % 计算各阈值的 PCK  
        for k = 1:numel(range)  
            d = squeeze(dist(1,jidx,:));  
            % dist is NaN if gt is missing; ignore dist in this case  
            pck(k,jidx) = 100*mean(d(~isnan(d))<=range(k));  
        end  
    end  
      
    % 计算上半身关键点的平均 PCK  
    for k = 1:numel(range)  
        d = reshape(squeeze(dist(1,7:12,:)),6*size(dist,3),1);  
        pck(k,end-1) = 100*mean(d(~isnan(d))<=range(k));  
    end  
      
    % 计算全身关键点的 PCK  
    for k = 1:numel(range)  
        d = reshape(squeeze(dist(1,:,:)),size(dist,2)*size(dist,3),1);  
        pck(k,end) = 100*mean(d(~isnan(d))<=range(k));  
    end  
    end  
      
  
---|---  
  
#### 2.PDJ - Percentage of Detected Joints

检测到的关键点比例.

    
    
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
    

|

    
    
    function [accs, range] = eval_pdj(pred, joints, reference_joints_pair, symmetry_joint_id, joint_name, eval_name)  
    % 设定选定的参考点：reference_joints_pair = [3, 10];     % 右肩点到左臀点  
    % symmetry_joint_id: 具有对称关系的关键点 ID  
    assert(numel(reference_joints_pair) == 2);  
    show_joint_ids = find(symmetry_joint_id >= 1:numel(symmetry_joint_id));   
    range = 0:0.01:0.5;  
      
    num = size(pred, 3);  
    assert(num >= 1);  
    % the number of joints  
    joint_n = size(joints, 2);  
      
    scale = zeros(1, num);  
    for ii = 1:num  
      scale(ii) = norm( joints(:,reference_joints_pair(1), ii) - joints(:,reference_joints_pair(2), ii) );  
      %scale(ii) = 100;  
    end  
      
    dists = zeros(num, joint_n);  
    for ii = 1:num  
      dists(ii,:) = sqrt(sum( (pred(:, :, ii) - joints(:,:,ii)).^2, 1 ));  
      dists(ii,:) = dists(ii,:) / scale(ii);  
    end  
      
    accs = zeros(numel(range), joint_n);  
    for ii = 1:numel(range)  
      accs(ii,:) = mean(dists <= range(ii),1);  
    end  
      
    accs = (accs + accs(:,symmetry_joint_id)) / 2;  
    accs = accs(:, show_joint_ids);  
    % print  
    fprintf('-------------- PDJ Evaluation ---------------n')  
    fprintf('Joints    '); fprintf('& %s ', joint_name{:}); fprintf('n');  
    sample_pdj_thresholds = [0.1, 0.2, 0.3, 0.4];  
    for ii = 1:length(sample_pdj_thresholds)  
      t = sample_pdj_thresholds(ii);  
      idx = (range == t);  
      fprintf('PDJ@%.2f  ', t); fprintf('& %.1f ', accs(idx,:)*100); fprintf('n');  
    end  
      
    % plot  
    line_width = 2;  
    p_color = {'g','y','b','r','c','k','m'};  
      
    % visualize  
    figure; hold on; grid on;  
    for ii = 1:numel(show_joint_ids)  
      plot(range, accs(:, ii), p_color{mod(ii, numel(p_color))+1}, 'linewidth', line_width);  
    end  
    leg_str = cell(numel(show_joint_ids), 1);  
    for ii = 1:numel(show_joint_ids)  
      leg_str{ii} = sprintf('%s', joint_name{ii});  
    end  
    h_leg = legend(leg_str, 'FontSize', 12);  
    set(h_leg, 'location', 'southeast', 'linewidth', 1);  
      
    axis([range(1),range(end), 0, 1]);  
    set(gca,'ytick', 0:0.1:1);  
    set(gca, 'linewidth', 1);  
      
    % --- titles  
    xlabel('Normalized  Threshold') % x-axis label  
    ylabel('Detection Rate') % y-axis label  
      
    title([eval_name ' PDJ']);  
    hold off;  
      
  
---|---