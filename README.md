```markdown
# 旋转离心坍落度测定仪中期改进方案系统框架图

```mermaid
graph TD
    %% 定义样式类
    classDef hardware fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#01579b;
    classDef legacySensors fill:#fff,stroke:#01579b,stroke-dasharray: 5 5,color:#01579b;
    classDef upCamera fill:#fff,stroke:#01579b,color:#01579b;
    classDef data_p fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#1b5e20;
    classDef feature_vec fill:#a5d6a7,stroke:#1b5e20,color:#fff;
    classDef algo fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#e65100;
    classDef script fill:#ffe0b2,stroke:#e65100,color:#e65100;
    classDef algo_model fill:#ffcc80,stroke:#e65100,color:#fff;
    classDef output_f fill:#fce4ec,stroke:#880e4f,stroke-width:2px,color:#880e4f;

    subgraph I_Phys [I. 物理输入与感知层]
        I_HW1[搅拌机内部水泥表面]
        I_HW2[电机驱动转盘]
        
        subgraph I_LegSens [短期方案保留]
            I_Sens1(多点高度传感器)
            I_Sens2(扭矩传感器)
            I_Sens3(温度传感器)
        end
        
        subgraph I_UpCam [中期硬件升级]
            I_Cam(立体视觉相机)
        end
    end
    
    subgraph II_Data [II. 数据层]
        II_HW[工控机/笔记本电脑]
        II_Unpack[数据解包与同步]
        II_Prepro[预处理/滤波降噪]
        II_Fusion[融合特征向量]:::feature_vec
    end
    
    subgraph III_Algo [III. 算法层]
        III_MLAlgo(XGBoost / LightGBM)
        III_Script[实时预测脚本]:::script
        III_AlgoModel[XGBoost融合校准模型]:::algo_model
    end
    
    subgraph IV_Output [IV. 输出与反馈层]
        IV_SlumpVal[预测坍落度值]
        IV_Shap(SHAP值特征重要性图)
    end

    %% 建立连接
    I_HW1 & I_HW2 & I_Cam --> II_HW
    II_HW --> II_Unpack --> II_Prepro --> II_Fusion
    II_Fusion --> III_AlgoModel
    III_MLAlgo --> III_Script --> III_AlgoModel
    III_AlgoModel --> IV_SlumpVal
    III_AlgoModel --> IV_Shap

    %% 应用样式
    class I_HW1,I_HW2 hardware;
    class I_LegSens,I_Sens1,I_Sens2,I_Sens3 legacySensors;
    class I_UpCam,I_Cam upCamera;
    class II_Data,II_HW,II_Unpack,II_Prepro data_p;
    class III_Algo algo;
    class IV_Output output_f;
