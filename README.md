graph TD
    %% 全局标题
    title[旋转离心坍落度测定仪中期改进方案系统框架图]
    
    %% 定义样式类
    classDef hardware fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#01579b;
    classDef legacySensors fill:#fff,stroke:#01579b,stroke-dasharray: 5 5,color:#01579b;
    classDef upCamera fill:#fff,stroke:#01579b,color:#01579b;
    
    classDef data_p fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#1b5e20;
    classDef feature_ext fill:#fff,stroke:#1b5e20,color:#1b5e20;
    classDef feature_vec fill:#a5d6a7,stroke:#1b5e20,color:#fff;
    
    classDef algo fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#e65100;
    classDef model_build fill:#fff,stroke:#e65100,color:#e65100;
    classDef script fill:#ffe0b2,stroke:#e65100,color:#e65100;
    classDef algo_model fill:#ffcc80,stroke:#e65100,color:#fff;
    
    classDef output_f fill:#fce4ec,stroke:#880e4f,stroke-width:2px,color:#880e4f;
    classDef explain fill:#fff,stroke:#880e4f,color:#880e4f;
    classDef output_val fill:#f8bbd0,stroke:#880e4f,color:#fff;

    %% 主层级结构
    
    subgraph I_Phys [I. 物理输入与感知层 (Hardware & Sensing)]
        I_HW1[搅拌机内部水泥表面]
        I_HW2[电机驱动转盘]
        
        subgraph I_LegSens [短期方案保留 (Legacy Sensors)]
            I_Sens1(多点高度传感器 - 激光/超声波)
            I_Sens2(扭矩传感器)
            I_Sens3(温度传感器)
        end
        
        subgraph I_UpCam [中期硬件升级 (Upgraded - RealSense)]
            I_Cam(立体视觉相机 - Intel RealSense D435i/D455)
        end
        
        %% 层内连接
        I_Cam -->|获取数据| I_HW1
        I_Cam -->|获取数据| I_HW2
        I_Sens1 -.-> I_HW1
        I_Sens2 -.-> I_HW2
        I_Sens3 -.-> I_HW1
        
        I_Cam -.-> I_Cam
    end
    
    subgraph II_Data [II. 数据层 (Data Processing - Python/Open3D)]
        II_HW[工控机/笔记本电脑]
        II_Unpack[数据解包与同步]
        II_Prepro[预处理/滤波降噪]
        
        subgraph II_FeatExt [特征提取 (Feature Extraction)]
            II_PhysFeat[物理特征特征向量]:::feature_vec
            II_ImgFeat[3D图像特征向量]:::feature_vec
            
            II_Phys1(高度差 ∆H)
            II_Phys2(扭矩 M)
            II_Phys3(扭矩 ∆M)
            II_Phys4(温度余量)
            
            II_Img1(表面高度统计)
            II_Img2(曲率特征)
            II_Img3(曲率特征)
            II_Img4(...)
            
            %% 特征提取内部连接
            II_PhysFeat -- |包含| II_Phys1
            II_PhysFeat -- |包含| II_Phys2
            II_PhysFeat -- |包含| II_Phys3
            II_PhysFeat -- |包含| II_Phys4
            
            II_ImgFeat -- |包含| II_Img1
            II_ImgFeat -- |包含| II_Img2
            II_ImgFeat -- |包含| II_Img3
            II_ImgFeat -- |包含| II_Img4
        end
        
        II_Fusion[融合特征向量]:::feature_vec
        
        %% 层内连接
        II_HW --> II_Unpack
        II_Unpack --> II_Prepro
        II_Prepro --> II_FeatExt
        II_FeatExt --> II_Fusion
        
        %% 确保特征向量并列进入融合向量
        II_PhysFeat -.-> II_Fusion
        II_ImgFeat -.-> II_Fusion
    end
    
    subgraph III_Algo [III. 算法层 (Machine Learning Model)]
        subgraph III_ModelBuild [模型建立 (Modeling - offline)]
            III_HistData(200组+历史样本数据)
            III_SlumpLabels(真实坍落度标签 - 标准筒试验)
            III_MLAlgo(XGBoost / LightGBM)
            III_ModelFile(训练好的模型文件 .json)
            
            %% 模型建立内部连接
            III_HistData --> III_MLAlgo
            III_SlumpLabels --> III_MLAlgo
            III_MLAlgo -- |训练| --> III_ModelFile
        end
        
        III_Script[实时预测脚本]:::script
        III_Implement[加入实现型]:::script
        
        III_AlgoModel[XGBoost融合校准模型]:::algo_model
        
        %% 层内连接
        III_ModelFile --> III_Script
        III_Implement --> III_AlgoModel
        III_Script -->|调用模型| III_AlgoModel
    end
    
    subgraph IV_Output [IV. 输出与反馈层 (Output & Feedback)]
        IV_SysDisp[现场系统显示/指导调整]
        IV_SlumpVal[预测坍落度值]:::output_val
        IV_Precision(精度: ±10-15mm)
        
        subgraph IV_Explain [解释性分析]:::explain
            IV_Shap(SHAP值特征重要性图)
        end
        
        %% 层内连接
        IV_SlumpVal -- |关联| IV_Precision
        IV_Explain -- |包含| IV_Shap
    end
    
    %% 层级间连接 (主要数据流和控制流)
    I_HW1 -->|传感器数据| II_HW
    I_HW2 -->|传感器数据| II_HW
    I_Cam -->|相机数据| II_HW
    I_Sens1 -.-> II_HW
    I_Sens2 -.-> II_HW
    I_Sens3 -.-> II_HW
    
    II_Fusion -->|特征向量| III_AlgoModel
    
    III_AlgoModel -->|预测值| IV_SlumpVal
    III_AlgoModel -->|模型参数| IV_Explain
    IV_SlumpVal -->|结果展示| IV_SysDisp

    %% 应用样式类
    I_HW1:::hardware
    I_HW2:::hardware
    I_LegSens:::legacySensors
    I_Sens1:::legacySensors
    I_Sens2:::legacySensors
    I_Sens3:::legacySensors
    I_UpCam:::upCamera
    I_Cam:::upCamera
    
    II_Data:::data_p
    II_HW:::data_p
    II_Unpack:::data_p
    II_Prepro:::data_p
    II_FeatExt:::feature_ext
    II_Phys1:::feature_ext
    II_Phys2:::feature_ext
    II_Phys3:::feature_ext
    II_Phys4:::feature_ext
    II_Img1:::feature_ext
    II_Img2:::feature_ext
    II_Img3:::feature_ext
    II_Img4:::feature_ext
    
    III_Algo:::algo
    III_ModelBuild:::model_build
    III_HistData:::model_build
    III_SlumpLabels:::model_build
    III_MLAlgo:::model_build
    III_ModelFile:::model_build
    
    IV_Output:::output_f
    IV_SysDisp:::output_f
    IV_Explain:::explain
    IV_Shap:::explain
    IV_Precision:::output_f
