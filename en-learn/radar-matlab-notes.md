# LiDAR 3D 目标追踪可视化

## 3D 目标追踪可视化（LiDAR 动态检测影院）

```matlab
% 1. 安全抓取全时间段的数据 
all_pts_matrix   = out.get('points'); 
all_boxes_matrix = out.get('targeBoxes'); 

if ~isnumeric(all_pts_matrix) 
    all_pts_matrix = all_pts_matrix.Values.Data; 
    all_boxes_matrix = all_boxes_matrix.Values.Data; 
end 

% 获取总帧数 (从点云的第三维获取最准) 
total_frames = size(all_pts_matrix, 3); 

% 检查 targetBoxes 的维度并强制对齐 
box_size = size(all_boxes_matrix); 
disp(['[检查数据] 原始目标框矩阵维度为: ', num2str(box_size)]); 

% 2. 创建 3D 播放动画窗口 
fig = figure('Name', 'LiDAR 3D 目标动态检测影院', 'Color', [0.1 0.1 0.1], 'MenuBar', 'figure', 'ToolBar', 'figure'); 
ax = axes('Parent', fig); 
rotate3d(fig, 'on'); 
disp('--> 正在启动动态流媒体播放，请点击黑窗并按住鼠标左键拖动以旋转视角...'); 

% 美化视角：【移到循环外面！】 
% 在进入循环前，只设定一次初始的斜俯视 3D 视角 
view(ax, [30, 30, 20]); 

% 3. 循环播放每一帧 
for f = 1:5:total_frames 
    if ~ishandle(fig), break; end 
    
    % 【新增】在刷新前，偷偷记住你当前用鼠标转动的 3D 视角角度！ 
    [current_az, current_el] = view(ax); 
    
    % 提取当前帧的点云 
    current_points = all_pts_matrix(:, :, f); 
    
    % 核心安全切片 
    if length(box_size) == 3 
        current_boxes = all_boxes_matrix(:, :, f); 
    elseif length(box_size) == 2 && box_size(1) == total_frames 
        row_data = all_boxes_matrix(f, :); 
        current_boxes = reshape(row_data, 50, 7); 
    else 
        error('目标框数据维度与帧数不匹配，请检查导出格式！'); 
    end 
    
    % 清除上一帧的画面 
    cla(ax); 
    
    % 绘制当前帧点云 
    ptCloud = pointCloud(current_points); 
    pcshow(ptCloud, 'MarkerSize', 30, 'Parent', ax); 
    hold(ax, 'on'); 
    
    % 【核心修复】立刻恢复你刚才用鼠标转动的视角，防止被 pcshow 重置！ 
    view(ax, [current_az, current_el]); 
    
    % 寻找当前帧中所有 Length > 0.1 米的真实目标 
    valid_idxs = find(current_boxes(:, 4) > 0.1); 
    
    % 在当前帧绘制所有检测到的 3D 框 
    for i = 1:length(valid_idxs) 
        idx = valid_idxs(i); 
        
        cx = current_boxes(idx, 1); 
        cy = current_boxes(idx, 2); 
        cz = current_boxes(idx, 3); 
        L  = current_boxes(idx, 4); 
        W  = current_boxes(idx, 5); 
        H  = current_boxes(idx, 6); 
        id = current_boxes(idx, 7); 
        
        if L <= 0.1 || W <= 0.1 || H <= 0.1 
            continue; 
        end 
        
        cuboid_9p = double([cx, cy, cz, L, W, H, 0, 0, 0]); 
        
        % 绘制绿色立体框 
        showShape('cuboid', cuboid_9p, 'Color', 'green', 'Opacity', 0.1, 'LineWidth', 2, 'Parent', ax); 
        
        % 标记黄色 ID（如果不想看它，把下面这行注掉即可） 
        text(ax, cx, cy, cz + H/2 + 0.4, ['ID: ' num2str(id)], ... 
            'Color', 'yellow', 'FontSize', 9, 'FontWeight', 'bold', 'HorizontalAlignment', 'center'); 
    end 
    
    % 更新标题 
    title(ax, ['激光雷达 3D 目标追踪流 - 当前第 ', num2str(f), ' / ', num2str(total_frames), ' 帧'], 'Color', 'white', 'FontSize', 12); 
    axis(ax, 'equal'); 
    set(ax, 'Color', [0.1 0.1 0.1], 'XColor', 'white', 'YColor', 'white', 'ZColor', 'white'); 
    
    pause(0.05); 
end
```

**代码说明：**
- **数据安全处理**：支持多种数据格式（直接矩阵或 `Values.Data` 结构）
- **维度自适应**：自动处理 `[50, 7, frames]` 和 `[frames, 350]` 两种存储格式
- **动态可视化**：实时渲染点云和3D目标框，支持鼠标旋转视角
- **目标过滤**：只显示 `Length > 0.1m` 的真实目标，过滤无效框
- **目标框格式**：`[cx, cy, cz, L, W, H, id]` - 中心点坐标 + 长宽高 + 目标ID

---

## 3D 目标追踪可视化（LiDAR 极致丝滑影院）

```matlab
% 1. 安全抓取全时间段的数据 
all_pts_matrix   = out.get('points'); 
all_boxes_matrix = out.get('targeBoxes'); 
if ~isnumeric(all_pts_matrix) 
    all_pts_matrix = all_pts_matrix.Values.Data; 
    all_boxes_matrix = all_boxes_matrix.Values.Data; 
end 
total_frames = size(all_pts_matrix, 3); 
box_size = size(all_boxes_matrix); 

% 2. 创建窗口 
fig = figure('Name', 'LiDAR 3D 极致丝滑影院', 'Color', [0.1 0.1 0.1]); 
ax = axes('Parent', fig); 
rotate3d(fig, 'on'); 

% --- 【核心改变：初始化第一帧】 --- 
f = 1; 
current_points = all_pts_matrix(:, :, f); 
ptCloud = pointCloud(current_points); 

% 拿到点云的句柄 hPc 
hPc = pcshow(ptCloud, 'MarkerSize', 30, 'Parent', ax); 
hold(ax, 'on'); 
view(ax, [30, 30, 20]); 
axis(ax, 'equal'); 
set(ax, 'Color', [0.1 0.1 0.1], 'XColor', 'white', 'YColor', 'white', 'ZColor', 'white'); 

% 初始化 3D 框和文字的句柄数组（假定最多 50 个目标） 
hBoxes = cell(50, 1); 
hTexts = cell(50, 1); 

% 3. 循环更新数据 
for f = 1:5:total_frames 
    if ~ishandle(fig), break; end 
    
    % 提取当前帧数据 
    current_points = all_pts_matrix(:, :, f); 
    if length(box_size) == 3 
        current_boxes = all_boxes_matrix(:, :, f); 
    elseif length(box_size) == 2 && box_size(1) == total_frames 
        row_data = all_boxes_matrix(f, :); 
        current_boxes = reshape(row_data, 50, 7); 
    end 
    
    % --- 【关键步骤 1：直接更新点云数据，不重绘】 --- 
    set(hPc, 'XData', current_points(:,1), 'YData', current_points(:,2), 'ZData', current_points(:,3)); 
    % 如果点云有颜色，也可以更新 CData: set(hPc, 'CData', current_colors); 
    
    % 寻找真实目标 
    valid_idxs = find(current_boxes(:, 4) > 0.1); 
    
    % --- 【关键步骤 2：更新或绘制 3D 框】 --- 
    % 先把上一帧的框和文字全部隐形/删除 
    for k = 1:length(hBoxes) 
        if ~isempty(hBoxes{k}) && ishandle(hBoxes{k}), delete(hBoxes{k}); end 
        if ~isempty(hTexts{k}) && ishandle(hTexts{k}), delete(hTexts{k}); end 
    end 
    
    % 绘制今年当前帧的新框 
    for i = 1:length(valid_idxs) 
        idx = valid_idxs(i); 
        cx = current_boxes(idx, 1); cy = current_boxes(idx, 2); cz = current_boxes(idx, 3); 
        L  = current_boxes(idx, 4); W  = current_boxes(idx, 5); H  = current_boxes(idx, 6); 
        id = current_boxes(idx, 7); 
        
        if L <= 0.1 || W <= 0.1 || H <= 0.1, continue; end 
        
        cuboid_9p = double([cx, cy, cz, L, W, H, 0, 0, 0]); 
        
        % 重新生成当前帧的形状并保存句柄 
        hBoxes{i} = showShape('cuboid', cuboid_9p, 'Color', 'green', 'Opacity', 0.1, 'LineWidth', 2, 'Parent', ax); 
        hTexts{i} = text(ax, cx, cy, cz + H/2 + 0.4, ['ID: ' num2str(id)], ... 
            'Color', 'yellow', 'FontSize', 9, 'FontWeight', 'bold', 'HorizontalAlignment', 'center'); 
    end 
    
    % 更新标题（标题更新不影响视角） 
    title(ax, ['激光雷达 3D 目标追踪 - 当前第 ', num2str(f), ' / ', num2str(total_frames), ' 帧'], 'Color', 'white', 'FontSize', 12); 
    
    pause(0.04); % 维持稳定帧率 
end
```

**代码说明：**
- **极致丝滑体验**：通过直接更新点云数据（`set(hPc, 'XData', ...)`）而非重绘，避免视角重置
- **句柄管理**：使用句柄数组管理3D框和文字，精确控制每个对象的生命周期
- **更高帧率**：`pause(0.04)` 比 `pause(0.05)` 更快，动画更流畅
- **资源安全**：每次循环结束时删除旧的框和文字，避免内存泄漏
