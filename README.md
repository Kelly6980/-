# -import networkx as nx
import numpy as np
import random
import matplotlib.pyplot as plt
from matplotlib.patches import PathPatch
from matplotlib.path import Path

# ====================== 1. 初始化参数 ======================
m0, n0, p0 = 32, 5, 40
stages = [
    {"name": "初始阶段", "s": 32, "m": 5, "r": 40},
    {"name": "第一阶段", "s": 64, "m": 10, "r": 80},
    {"name": "第二阶段", "s": 96, "m": 15, "r": 120},
    {"name": "第三阶段", "s": 128, "m": 20, "r": 160},
    {"name": "第四阶段", "s": 160, "m": 25, "r": 200}
]
s = 3
t = 1

# 初始化图
G = nx.Graph()

# ---------------------- 1.1 添加初始节点 ----------------------
suppliers = [f"S_{i}" for i in range(m0)]
manufacturers = [f"M_{i}" for i in range(n0)]
retailers = [f"R_{i}" for i in range(p0)]

G.add_nodes_from(suppliers, type='S')
G.add_nodes_from(manufacturers, type='M')
G.add_nodes_from(retailers, type='R')

# 初始连边
for s_node in suppliers:
    connect_count = random.randint(1, 2)
    m_nodes = random.sample(manufacturers, connect_count)
    for m_node in m_nodes:
        G.add_edge(s_node, m_node)

for m_node in manufacturers:
    connect_count = random.randint(5, 10)
    r_nodes = random.sample(retailers, connect_count)
    for r_node in r_nodes:
        G.add_edge(m_node, r_node)

print(f"初始阶段构建完成，节点数：{G.number_of_nodes()}, 边数：{G.number_of_edges()}")

# ====================== 2. 网络演化（分阶段记录） ======================
stage_graphs = []
stage_graphs.append((G.copy(), "初始阶段"))

# 2.1 新增供应商
new_supplier_id = m0
current_s = m0
while current_s < stages[-1]["s"]:
    new_node = f"S_{new_supplier_id}"
    G.add_node(new_node, type='S')
    m_nodes = [n for n, d in G.nodes(data=True) if d['type'] == 'M']
    edges_added = 0

    if edges_added < t and len(m_nodes) > 0:
        target = random.choice(m_nodes)
        if not G.has_edge(new_node, target):
            G.add_edge(new_node, target)
            edges_added += 1

    while edges_added < s and len(m_nodes) > 0:
        degrees = np.array([G.degree(n) for n in m_nodes])
        if degrees.sum() == 0:
            target = random.choice(m_nodes)
        else:
            probs = degrees / degrees.sum()
            target = np.random.choice(m_nodes, p=probs)
        if not G.has_edge(new_node, target):
            G.add_edge(new_node, target)
            edges_added += 1

    new_supplier_id += 1
    current_s += 1

    for stage in stages[1:]:
        if current_s == stage["s"]:
            stage_graphs.append((G.copy(), stage["name"]))

# 2.2 新增制造商
new_manufacturer_id = n0
current_m = n0
while current_m < stages[-1]["m"]:
    new_node = f"M_{new_manufacturer_id}"
    G.add_node(new_node, type='M')
    s_nodes = [n for n, d in G.nodes(data=True) if d['type'] == 'S']
    r_nodes = [n for n, d in G.nodes(data=True) if d['type'] == 'R']
    all_partners = s_nodes + r_nodes
    edges_added = 0

    if edges_added < t and len(all_partners) > 0:
        target = random.choice(all_partners)
        if not G.has_edge(new_node, target):
            G.add_edge(new_node, target)
            edges_added += 1

    while edges_added < s and len(all_partners) > 0:
        degrees = np.array([G.degree(n) for n in all_partners])
        if degrees.sum() == 0:
            target = random.choice(all_partners)
        else:
            probs = degrees / degrees.sum()
            target = np.random.choice(all_partners, p=probs)
        if not G.has_edge(new_node, target):
            G.add_edge(new_node, target)
            edges_added += 1

    new_manufacturer_id += 1
    current_m += 1

    for stage in stages[1:]:
        if current_m == stage["m"] and not any(name == stage["name"] for _, name in stage_graphs):
            stage_graphs.append((G.copy(), stage["name"]))

# 2.3 新增销售商
new_retailer_id = p0
current_r = p0
while current_r < stages[-1]["r"]:
    new_node = f"R_{new_retailer_id}"
    G.add_node(new_node, type='R')
    m_nodes = [n for n, d in G.nodes(data=True) if d['type'] == 'M']
    edges_added = 0

    if edges_added < t and len(m_nodes) > 0:
        target = random.choice(m_nodes)
        if not G.has_edge(new_node, target):
            G.add_edge(new_node, target)
            edges_added += 1

    while edges_added < s and len(m_nodes) > 0:
        degrees = np.array([G.degree(n) for n in m_nodes])
        if degrees.sum() == 0:
            target = random.choice(m_nodes)
        else:
            probs = degrees / degrees.sum()
            target = np.random.choice(m_nodes, p=probs)
        if not G.has_edge(new_node, target):
            G.add_edge(new_node, target)
            edges_added += 1

    new_retailer_id += 1
    current_r += 1

    for stage in stages[1:]:
        if current_r == stage["r"] and not any(name == stage["name"] for _, name in stage_graphs):
            stage_graphs.append((G.copy(), stage["name"]))

print(f"网络演化完成，节点数：{G.number_of_nodes()}, 边数：{G.number_of_edges()}")

# ====================== 3. 美化绘图：弯曲边函数 ======================
def draw_curved_edges(ax, G, pos, curvature=0.2, alpha=0.4, color='gray', lw=0.6):
    for u, v in G.edges():
        x1, y1 = pos[u]
        x2, y2 = pos[v]
        cx = (x1 + x2) / 2 + curvature * (y2 - y1)
        cy = (y1 + y2) / 2 - curvature * (x2 - x1)
        path_data = [Path.MOVETO, (x1, y1), Path.CURVE3, (cx, cy), Path.LINETO, (x2, y2)]
        codes, verts = zip(*[(path_data[i], path_data[i+1]) for i in range(0, len(path_data), 2)])
        path = Path(verts, codes)
        patch = PathPatch(path, facecolor='none', edgecolor=color, alpha=alpha, linewidth=lw)
        ax.add_patch(patch)

# ====================== 4. 分阶段可视化（最终美化版 + 图例） ======================
plt.rcParams['figure.dpi'] = 300
fig, axes = plt.subplots(3, 2, figsize=(15, 19))
axes = axes.flatten()

# 定义配色（高级论文色）
colors = {
    'S': '#4285F4',   # 供应商：蓝
    'M': '#FBBC05',   # 制造商：黄
    'R': '#34A853'    # 销售商：绿
}

for idx, (g, title) in enumerate(stage_graphs):
    ax = axes[idx]

    # 1. 删除孤立节点
    g_clean = g.copy()
    g_clean.remove_nodes_from([n for n, d in g_clean.degree() if d == 0])

    # 2. Gephi Fruchterman-Reingold 布局
    pos = nx.spring_layout(g_clean, k=0.28, iterations=120, seed=42)

    # 3. 按类型分组（用于图例）
    S_nodes = [n for n, d in g_clean.nodes(data=True) if d['type'] == 'S']
    M_nodes = [n for n, d in g_clean.nodes(data=True) if d['type'] == 'M']
    R_nodes = [n for n, d in g_clean.nodes(data=True) if d['type'] == 'R']

    # 4. 绘制弯曲边
    draw_curved_edges(ax, g_clean, pos, curvature=0.22, alpha=0.35, lw=0.5)

    # 5. 绘制节点（带白色描边）
    nx.draw_networkx_nodes(g_clean, pos, nodelist=S_nodes, node_color=colors['S'], node_size=32, alpha=0.94, edgecolors='white', linewidths=0.7, ax=ax)
    nx.draw_networkx_nodes(g_clean, pos, nodelist=M_nodes, node_color=colors['M'], node_size=36, alpha=0.94, edgecolors='white', linewidths=0.7, ax=ax)
    nx.draw_networkx_nodes(g_clean, pos, nodelist=R_nodes, node_color=colors['R'], node_size=32, alpha=0.94, edgecolors='white', linewidths=0.7, ax=ax)

    # 6. 标题
    ax.set_title(f'{title}\n有效节点：{g_clean.number_of_nodes()} | 边数：{g_clean.number_of_edges()}', fontsize=12, pad=12)
    ax.axis('off')

# 隐藏多余子图
for j in range(len(stage_graphs), len(axes)):
    axes[j].axis('off')

# ====================== 5. 添加全局图例 ======================
from matplotlib.patches import Patch
legend_elements = [
    Patch(facecolor=colors['S'], edgecolor='white', linewidth=0.7, label='Supplier 供应商'),
    Patch(facecolor=colors['M'], edgecolor='white', linewidth=0.7, label='Manufacturer 制造商'),
    Patch(facecolor=colors['R'], edgecolor='white', linewidth=0.7, label='Retailer 销售商')
]
fig.legend(handles=legend_elements, loc='upper center', ncol=3, bbox_to_anchor=(0.5, 0.985), fontsize=13, frameon=False)

plt.suptitle('新能源汽车电池供应链网络动态演化过程', fontsize=18, y=0.96)
plt.tight_layout()
plt.subplots_adjust(top=0.93)
plt.show()

# ====================== 6. 网络特征输出 ======================
print("\n===== 各阶段网络特征对比 =====")
for g, title in stage_graphs:
    degrees = [d for n, d in g.degree()]
    print(f"{title}: 节点={g.number_of_nodes()}, 边={g.number_of_edges()}, 平均度={np.mean(degrees):.2f}")
