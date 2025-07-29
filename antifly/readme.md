

## **1\. 项目目标与核心技术栈**

### **1.1 项目目标**

在单台MacBook上，构建一个完整的“查打一体”低空防御系统仿真环境。该环境将模拟从传感器探测、数据融合、AI识别、指挥决策到指令处置的全过程，为算法验证、战术推演和操作员培训提供一个高保真、低成本的开发与测试平台。

### **1.2 核心技术栈**

为确保跨平台兼容性、模块化和社区支持，我们选择以下基于Python的开源技术栈：

* **容器化环境:** **Docker Desktop for Mac** 1。使用  
  docker-compose编排多容器应用，为每个系统模块（传感器、指控平台等）提供隔离且一致的运行环境，极大简化了部署和依赖管理 2。  
* **三维物理仿真引擎:** **Gazebo** 5。一个强大的开源机器人仿真平台，用于构建逼真的城市三维环境（“城市峡谷”），并模拟无人机的物理飞行特性和传感器行为。  
* **机器人操作系统:** **ROS 2 (Robot Operating System 2\)**。作为各仿真模块之间通信的“神经网络”，其发布/订阅模式非常适合解耦系统中的各个组件（如传感器数据到指控平台）。  
* **核心编程语言:** **Python 3.9+**。凭借其在数据科学、机器学习和Web开发领域的强大生态，成为整个项目的核心语言 6。  
* **指控平台后端:** **FastAPI**。一个现代、高性能的Python Web框架，用于构建“御空”平台的后端服务。  
* **指控平台前端/GIS可视化:** **Streamlit** \+ **Pydeck**。Streamlit用于快速构建交互式Web应用界面，Pydeck（基于deck.gl）则提供了强大的3D地理空间数据可视化能力，可直接在浏览器中渲染城市模型和无人机航迹 9。  
* **AI视觉识别:** **PyTorch** \+ **YOLOv8**。使用YOLOv8这一先进的目标检测模型，对仿真摄像头传输的视频流进行实时无人机识别 11。  
* **数据融合算法:** **PyKalman** / **NumPy**。使用这些库实现卡尔曼滤波器（Kalman Filter），对来自不同模拟传感器的数据进行融合，生成稳定、精确的目标航迹 16。

## **2\. 分阶段实施路线图**

### **阶段一：基础环境搭建 (Crawl)**

此阶段的目标是为整个项目奠定坚实的基础。

1. **安装核心软件:**  
   * 在MacBook上安装 **Docker Desktop** 1。  
   * 安装 **Visual Studio Code** 并配置其 **Dev Containers** 扩展，以便在Docker容器内进行开发，保持本地环境的整洁 20。  
2. **项目结构初始化:**  
   * 使用git创建项目仓库。  
   * 建立清晰的目录结构，例如：  
     /c-uas-simulation

|-- docker-compose.yml  
|-- /simulation\_ws \# ROS 2 & Gazebo工作空间  
|-- /sensor\_simulators \# 各传感器模拟器模块  
| |-- /radar\_sim  
| |-- /rf\_sim  
| |-- /acoustic\_sim  
|-- /yukong\_c2\_platform \# “御空”指控平台  
| |-- /backend  
| |-- /frontend  
|-- /data \# 3D模型、地图数据、AI模型等  
\`\`\`  
3\. 创建docker-compose.yml: 定义项目的核心服务框架，包括Gazebo仿真服务、各个传感器服务以及“御空”指控平台的前后端服务 2。

### **阶段二：构建仿真世界 (Walk)**

此阶段专注于创建逼真的“城市峡谷”作战环境。

1. **搭建城市环境:**  
   * 在Gazebo中，使用开源的3D城市模型（或自行创建简化的建筑模型）搭建一个代表“政府要地/城市中心”的场景。重点是模拟出能导致信号遮挡和多径效应的“城市峡谷”环境。  
2. **创建无人机模型:**  
   * 在Gazebo中定义一个或多个无人机模型（如四旋翼无人机）。为其配置基础的物理属性（质量、惯性）和飞行控制器插件，使其能够响应ROS 2的指令。  
3. **定义威胁航线:**  
   * 使用ROS 2编写一个“航线发布器”节点，该节点可以发布一系列航点，引导无人机模型在仿真环境中按预设路径飞行（例如，低空快速接近、在关键建筑上空悬停等）。

### **阶段三：模拟感知层数据生成 (Walk/Run)**

此阶段的目标是让虚拟传感器“活”起来，产生模拟真实世界的数据流。所有传感器模拟器都将作为独立的Docker容器运行，通过ROS 2订阅Gazebo中的无人机状态（位置、速度等），并发布模拟的传感器数据。

1. **雷达模拟器 (radar\_sim):**  
   * **实现：** 使用Python编写一个ROS 2节点。订阅无人机真实状态。  
   * **数据生成：** 基于无人机的位置、速度和尺寸，模拟生成雷达探测点迹。关键是加入噪声：  
     * **位置噪声：** 在真实坐标上增加一个符合高斯分布的随机误差。  
     * **虚警：** 随机生成一些“鸟类”或“杂波”的虚假目标点迹 21。  
     * **漏报：** 根据无人机与建筑物的相对位置，模拟因遮挡导致的探测丢失。  
   * **输出：** 通过ROS 2话题发布包含\`\`等信息的探测数据流。可以使用RadarSimPy等库来辅助生成更真实的雷达信号特性 22。  
2. **射频探测模拟器 (rf\_sim):**  
   * **实现：** 编写一个ROS 2节点，订阅无人机状态。  
   * **数据生成：** 模拟RF探测器的输出。根据预设的无人机型号（如“大疆御3”），生成包含\`\`的JSON数据。信号强度应与无人机距离成反比。模拟城市电磁干扰，引入随机的信号丢失或识别错误 21。  
   * **输出：** 通过ROS 2话题发布模拟的RF探测日志。  
3. **光电/红外(EO/IR)模拟器:**  
   * **实现：** 这部分直接利用Gazebo的内置功能。在仿真环境中创建一个或多个可由ROS 2控制的云台摄像机模型。  
   * **数据生成：**  
     * **EO (可见光):** Gazebo摄像机插件可以直接发布标准的ROS 2图像话题（sensor\_msgs/Image）。  
     * **IR (红外):** 通过图像处理节点订阅可见光图像，将其转换为灰度图并应用热成像伪彩色映射，模拟红外热成像效果。  
   * **输出：** 发布实时的视频流话题。  
4. **声学探测模拟器 (acoustic\_sim):**  
   * **实现：** 类似RF模拟器，编写一个ROS 2节点。  
   * **数据生成：** 根据无人机距离和类型，生成\`\`数据。加入背景噪声模型，当无人机距离远或环境噪声大时，降低探测概率 21。  
   * **输出：** 通过ROS 2话题发布模拟的声学警报。

### **阶段四：“御空”指控平台软件实现 (Run)**

这是项目的核心，将所有模拟数据汇集于此，进行处理、分析、可视化和决策。

1. **后端服务 (yukong\_c2\_platform/backend):**  
   * **架构：** 使用FastAPI框架构建，运行在独立的Docker容器中。通过ROS 2客户端库订阅所有传感器模拟器发布的数据话题。  
   * **数据融合模块:**  
     * **核心算法：** 实现一个**扩展卡尔曼滤波器 (EKF)** 16。  
     * **输入：** 接收来自雷达、RF、光电（经AI识别后）等多个传感器的、带有不同噪声和更新频率的异步数据。  
     * **处理：** 当新的传感器数据到达时，执行EKF的“预测”和“更新”步骤，对目标的运动状态（位置、速度）进行最优估计。该模块需要能够处理数据丢失（当传感器被遮挡时仅执行预测）和数据关联（确认不同传感器的探测指向同一目标）的复杂情况。  
     * **输出：** 生成一个统一、平滑、高置信度的目标航迹列表。  
   * **AI识别模块:**  
     * **模型：** 加载一个在无人机、鸟类等数据集上预训练或微调过的**YOLOv8模型** 11。  
     * **输入：** 订阅来自EO/IR模拟器的视频流。  
     * **处理：** 对每一帧图像进行推理，检测图像中的无人机，并输出其边界框（bounding box）和置信度。  
     * **输出：** 将识别结果（如“发现大疆无人机，位置\[x,y\]”）作为一路新的观测数据，输入到数据融合模块。  
   * **威胁评估与处置决策模块:**  
     * **规则引擎：** 实现一个简单的规则引擎。例如：IF (目标航迹进入核心禁飞区 AND 目标类型 \== '无人机' AND 威胁等级 \> 阈值) THEN 触发高级警报。  
     * **处置推荐：** 根据目标类型和威胁等级，向前端推荐处置方案。例如，对于已识别型号的大疆无人机，优先推荐“协议接管”；对于未知型号，推荐“定向干扰” 25。  
   * **Websocket接口:** 使用FastAPI的Websocket功能，将处理后的统一态势（所有目标航迹、警报等）实时推送到前端。  
2. **前端仪表盘 (yukong\_c2\_platform/frontend):**  
   * **架构：** 使用Streamlit构建一个单页Web应用。  
   * **3D态势显示:**  
     * 使用**Pydeck**创建一个3D地图视图，加载城市建筑物的GeoJSON或3D Tiles模型 9。  
     * 通过Websocket从后端接收实时航迹数据，并在3D地图上动态渲染无人机图标、历史轨迹线和预测路径。  
     * 使用不同颜色或图标区分目标类型（无人机、鸟）和威胁等级（红、黄、绿）。  
   * **交互功能:**  
     * **目标列表：** 显示所有当前跟踪目标的详细信息（ID、类型、高度、速度）。  
     * **警报窗口：** 当后端发出警报时，弹出醒目的通知。  
     * **处置面板：** 点击某个高威胁目标后，显示后端推荐的处置选项（如“协议接管”、“定向干扰”）。点击按钮后，通过Websocket向后端发送处置指令。

### **阶段五：端到端集成与测试 (Fly)**

1. **指令闭环:**  
   * 当用户在前端点击“协议接管”按钮时，后端服务通过ROS 2发布一个处置指令。  
   * 仿真环境中的无人机模型节点订阅该指令，并执行相应的行为模拟，如改变航向，飞往预设的“安全降落区”。  
2. **全系统启动:**  
   * 在项目根目录运行 docker-compose up。这将一次性启动Gazebo仿真环境、所有传感器模拟器以及“御空”指控平台的前后端服务。  
3. **测试与验证:**  
   * 打开浏览器访问Streamlit前端地址。  
   * 观察无人机在3D城市中飞行，同时在指控平台上看到由多个模拟传感器生成的、经过融合后的平滑航迹。  
   * 验证当无人机飞入禁区时，系统是否能正确评估威胁并发出警报。  
   * 测试点击处置按钮后，仿真世界中的无人机是否按预期做出反应。

通过以上五个阶段，您将在您的MacBook上成功构建一个功能全面、逻辑闭环的低空防御模拟仿真系统。这个系统不仅是对真实世界复杂系统的一次高保真复现，更是一个宝贵的开发、测试和演练平台。

#### **Works cited**

1. Install Docker Desktop on Mac \- Docker Docs, accessed July 5, 2025, [https://docs.docker.com/desktop/setup/install/mac-install/](https://docs.docker.com/desktop/setup/install/mac-install/)  
2. How to set up Docker development environment on macOS or Windows \- LabEx, accessed July 5, 2025, [https://labex.io/tutorials/docker-how-to-set-up-docker-development-environment-on-macos-or-windows-411603](https://labex.io/tutorials/docker-how-to-set-up-docker-development-environment-on-macos-or-windows-411603)  
3. The best way to use docker with Python for development environments \- Reddit, accessed July 5, 2025, [https://www.reddit.com/r/docker/comments/1b768ft/the\_best\_way\_to\_use\_docker\_with\_python\_for/](https://www.reddit.com/r/docker/comments/1b768ft/the_best_way_to_use_docker_with_python_for/)  
4. How to Create a Great Local Python Development Environment with Docker \- YouTube, accessed July 5, 2025, [https://www.youtube.com/watch?v=6OxqiEeCvMI\&pp=0gcJCdgAo7VqN5tD](https://www.youtube.com/watch?v=6OxqiEeCvMI&pp=0gcJCdgAo7VqN5tD)  
5. Gazebo, accessed July 5, 2025, [https://gazebosim.org/](https://gazebosim.org/)  
6. cocotb | Python verification framework, accessed July 5, 2025, [https://www.cocotb.org/](https://www.cocotb.org/)  
7. NVIDIA/warp: A Python framework for accelerated simulation, data generation and spatial computing. \- GitHub, accessed July 5, 2025, [https://github.com/NVIDIA/warp](https://github.com/NVIDIA/warp)  
8. Simulation Testbeds and Frameworks for UAV Performance Evaluation \- NSF-PAR, accessed July 5, 2025, [https://par.nsf.gov/servlets/purl/10297241](https://par.nsf.gov/servlets/purl/10297241)  
9. How to create a 3D geospatial dashboard with Python, Streamlit and Pydeck \- Medium, accessed July 5, 2025, [https://medium.com/@agiraldoal/how-to-create-a-3d-geospatial-dashboard-with-python-streamlit-and-pydeck-c1f2cc3c2cf4](https://medium.com/@agiraldoal/how-to-create-a-3d-geospatial-dashboard-with-python-streamlit-and-pydeck-c1f2cc3c2cf4)  
10. kepler.gl, accessed July 5, 2025, [https://kepler.gl/](https://kepler.gl/)  
11. Realtime Drone Detection Using YOLOv8 and TensorFlow.JS \- Journal of Engineering Sciences, accessed July 5, 2025, [https://jespublication.com/uploads/2024-V15I2032.pdf](https://jespublication.com/uploads/2024-V15I2032.pdf)  
12. doguilmak/Drone-Detection-YOLOv8x: This repository provides a dataset and model for real-time drone detection using YOLOv8, contributing to enhanced security and privacy protection. Join us in advancing drone detection technology for safer environments. \- GitHub, accessed July 5, 2025, [https://github.com/doguilmak/Drone-Detection-YOLOv8x](https://github.com/doguilmak/Drone-Detection-YOLOv8x)  
13. EDGS-YOLOv8: An Improved YOLOv8 Lightweight UAV Detection Model \- MDPI, accessed July 5, 2025, [https://www.mdpi.com/2504-446X/8/7/337](https://www.mdpi.com/2504-446X/8/7/337)  
14. DRBD-YOLOv8: A Lightweight and Efficient Anti-UAV Detection Model \- PMC, accessed July 5, 2025, [https://pmc.ncbi.nlm.nih.gov/articles/PMC11598377/](https://pmc.ncbi.nlm.nih.gov/articles/PMC11598377/)  
15. YOLOv8-Based Drone Detection: Performance Analysis and Optimization \- MDPI, accessed July 5, 2025, [https://www.mdpi.com/2073-431X/13/9/234](https://www.mdpi.com/2073-431X/13/9/234)  
16. Sensor Fusion With Kalman Filter. Introduction | by Satya \- Medium, accessed July 5, 2025, [https://medium.com/@satya15july\_11937/sensor-fusion-with-kalman-filter-c648d6ec2ec2](https://medium.com/@satya15july_11937/sensor-fusion-with-kalman-filter-c648d6ec2ec2)  
17. Implementation of Extended Kalman Filter using Python \- Reddit, accessed July 5, 2025, [https://www.reddit.com/r/Python/comments/7vo1mx/implementation\_of\_extended\_kalman\_filter\_using/](https://www.reddit.com/r/Python/comments/7vo1mx/implementation_of_extended_kalman_filter_using/)  
18. sharathsrini/Kalman-Filter-for-Sensor-Fusion: A Sensor Fusion Algorithm that can predict a State Estimate and Update if it is uncertain \- GitHub, accessed July 5, 2025, [https://github.com/sharathsrini/Kalman-Filter-for-Sensor-Fusion](https://github.com/sharathsrini/Kalman-Filter-for-Sensor-Fusion)  
19. Paul Balzer \- IPython and Sympy to Develop a Kalman Filter for Multisensor Data Fusion, accessed July 5, 2025, [https://www.youtube.com/watch?v=XSRr2HHedrY](https://www.youtube.com/watch?v=XSRr2HHedrY)  
20. Setting Up a Great Python Dev Environment with Docker, accessed July 5, 2025, [https://dev.to/njoguu/setting-up-a-great-python-dev-environment-with-docker-2b01](https://dev.to/njoguu/setting-up-a-great-python-dev-environment-with-docker-2b01)  
21. 国内外反无人机技术发展综述- 安全内参| 决策者的网络安全知识库, accessed July 4, 2025, [https://www.secrss.com/articles/76379](https://www.secrss.com/articles/76379)  
22. radarsimx/radarsimpy: Radar Simulator built with Python and C++ \- GitHub, accessed July 5, 2025, [https://github.com/radarsimx/radarsimpy](https://github.com/radarsimx/radarsimpy)  
23. 反无人机作战：全球攻防新战场 \- 新华网, accessed July 4, 2025, [http://www.xinhuanet.com/milpro/20250318/020a9e13a2be4d3496da460966ef4606/c.html](http://www.xinhuanet.com/milpro/20250318/020a9e13a2be4d3496da460966ef4606/c.html)  
24. 反无人蜂群技术研究现状, accessed July 4, 2025, [https://bzxb.cqut.edu.cn/download.aspx?type=paper\&id=10290](https://bzxb.cqut.edu.cn/download.aspx?type=paper&id=10290)  
25. ​立体化低慢小飞行物探测与防御系统设计及应用- 安全内参| 决策者的 ..., accessed July 4, 2025, [https://www.secrss.com/articles/14801](https://www.secrss.com/articles/14801)