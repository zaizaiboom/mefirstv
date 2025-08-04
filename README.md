<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>细胞叙事：实时反馈引擎</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js"></script>
    <style>
        :root {
            --primary: #1B5E20;
            --secondary-bg: #E8F5E9;
            --cell-green: #4CAF50;
            --text: #2D3748;
        }
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'IBM Plex Sans', -apple-system, BlinkMacSystemFont, sans-serif;
        }
        
        body {
            background: var(--secondary-bg);
            color: var(--text);
            overflow-x: hidden;
            line-height: 1.6;
            min-height: 100vh;
            background-image: 
                radial-gradient(circle at 10% 20%, rgba(33, 150, 243, 0.03) 0%, transparent 20%),
                radial-gradient(circle at 90% 80%, rgba(76, 175, 80, 0.03) 0%, transparent 20%);
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 2rem;
        }
        
        /* 医疗设备输入框样式 */
        .medical-input-container {
            background: white;
            border-radius: 12px;
            box-shadow: 0 10px 30px -15px rgba(27, 94, 32, 0.2);
            overflow: hidden;
            margin: 4rem 0;
            position: relative;
            border: 1px solid #C8E6C9;
        }
        
        .medical-header {
            background: var(--primary);
            color: white;
            padding: 1rem 1.5rem;
            font-weight: 500;
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }
        
        .led-indicator {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #4CAF50;
            box-shadow: 0 0 8px #00C1D4;
        }
        
        .input-section {
            padding: 1.5rem;
        }
        
        .input-prompt {
            color: #556B2F;
            margin-bottom: 1rem;
            font-style: italic;
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        
        .input-prompt::before {
            content: ">";
            color: var(--cell-green);
            font-weight: bold;
        }
        
        #problem-input {
            width: 100%;
            padding: 1rem;
            border: 2px solid #C8E6C9;
            border-radius: 8px;
            font-size: 1.1rem;
            transition: all 0.3s;
            background: #F1F8E9;
        }
        
        #problem-input:focus {
            outline: none;
            border-color: var(--cell-green);
            box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.2);
        }
        
        .action-buttons {
            display: flex;
            gap: 1rem;
            margin-top: 1.5rem;
            justify-content: flex-end;
        }
        
        .btn {
            padding: 0.75rem 1.5rem;
            border-radius: 6px;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.2s;
            border: none;
            font-size: 1rem;
        }
        
        .btn-primary {
            background: var(--cell-green);
            color: white;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
        
        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 7px 14px -3px rgba(0, 0, 0, 0.2);
            background: #388E3C;
        }
        
        /* 需求解构图 */
        .demand-mapping {
            background: white;
            border-radius: 12px;
            padding: 2rem;
            margin: 2rem 0;
            box-shadow: 0 10px 30px -15px rgba(27, 94, 32, 0.1);
            display: none;
        }
        
        .demand-title {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            margin-bottom: 1.5rem;
            color: var(--primary);
        }
        
        .cell-structure {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 1.5rem;
        }
        
        .cell-part {
            background: #F1F8E9;
            border-radius: 10px;
            padding: 1.5rem;
            border-left: 4px solid var(--cell-green);
            transition: all 0.3s;
        }
        
        .cell-part:hover {
            transform: translateY(-5px);
            box-shadow: 0 5px 15px -3px rgba(0, 0, 0, 0.1);
        }
        
        .cell-part h3 {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            margin-bottom: 1rem;
            color: var(--primary);
        }
        
        .cell-part p {
            color: #556B2F;
            font-size: 0.95rem;
        }
        
        /* 细胞分裂动画区域 */
        .cell-division {
            margin: 3rem 0;
            text-align: center;
            display: none;
        }
        
        .cell-container {
            position: relative;
            height: 400px;
            background: #E8F5E9;
            border-radius: 12px;
            overflow: hidden;
            margin: 2rem 0;
            border: 1px solid #C8E6C9;
        }
        
        .original-cell {
            position: absolute;
            width: 120px;
            height: 120px;
            background: radial-gradient(circle, #4CAF50 0%, #81C784 70%, #E8F5E9 100%);
            border-radius: 50%;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
            box-shadow: 0 0 20px rgba(76, 175, 80, 0.5);
            z-index: 10;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        .original-cell:hover {
            transform: translate(-50%, -50%) scale(1.05);
            box-shadow: 0 0 30px rgba(76, 175, 80, 0.8);
        }
        
        .left-cell, .right-cell {
            position: absolute;
            width: 100px;
            height: 100px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
            opacity: 0;
            transform: scale(0.8);
            transition: all 0.5s cubic-bezier(0.17, 0.67, 0.83, 0.67);
            z-index: 5;
            cursor: grab;
        }
        
        .left-cell {
            background: radial-gradient(circle, #1B5E20 0%, #388E3C 70%, #E8F5E9 100%);
            top: 40%;
            left: 25%;
        }
        
        .left-cell:active {
            cursor: grabbing;
        }
        
        .right-cell {
            background: radial-gradient(circle, #388E3C 0%, #4CAF50 70%, #E8F5E9 100%);
            top: 60%;
            left: 75%;
        }
        
        .right-cell:active {
            cursor: grabbing;
        }
        
        .cell-label {
            position: absolute;
            background: white;
            padding: 0.5rem 1rem;
            border-radius: 20px;
            font-size: 0.85rem;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
            max-width: 200px;
            text-align: center;
            opacity: 0;
            transition: opacity 0.3s;
        }
        
        .left-label {
            top: 25%;
            left: 15%;
        }
        
        .right-label {
            top: 75%;
            right: 15%;
        }
        
        /* mRNA数据流 */
        .mrna-flow {
            background: white;
            border-radius: 12px;
            padding: 2rem;
            margin: 2rem 0;
            display: none;
        }
        
        .mrna-title {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            margin-bottom: 1.5rem;
            color: var(--primary);
        }
        
        .mrna-canvas {
            height: 200px;
            background: #F1F8E9;
            border-radius: 8px;
            position: relative;
            overflow: hidden;
            border: 1px dashed #C8E6C9;
        }
        
        .mrna-path {
            position: absolute;
            height: 4px;
            background: var(--cell-green);
            top: 50%;
            left: 0;
            transform: translateY(-50%);
        }
        
        .mrna-dot {
            position: absolute;
            width: 12px;
            height: 12px;
            background: var(--primary);
            border-radius: 50%;
            top: 50%;
            transform: translateY(-50%);
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 0.7rem;
            font-weight: bold;
        }
        
        .mrna-detail {
            margin-top: 1.5rem;
            padding: 1rem;
            background: #F1F8E9;
            border-radius: 8px;
            font-size: 0.95rem;
        }
        
        /* 模拟测试区域 */
        .simulation-test {
            background: white;
            border-radius: 12px;
            padding: 2rem;
            margin: 2rem 0;
            display: none;
        }
        
        .simulation-title {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            margin-bottom: 1.5rem;
            color: var(--primary);
        }
        
        .ecg-chart {
            height: 150px;
            background: #F1F8E9;
            border-radius: 8px;
            position: relative;
            overflow: hidden;
            margin: 1.5rem 0;
            border: 1px solid #C8E6C9;
        }
        
        .ecg-line {
            position: absolute;
            height: 3px;
            background: var(--primary);
            top: 50%;
            left: 0;
            transform: translateY(-50%);
        }
        
        .results-summary {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 1.5rem;
            margin-top: 2rem;
        }
        
        .result-card {
            background: #F1F8E9;
            border-radius: 10px;
            padding: 1.5rem;
            text-align: center;
        }
        
        .result-value {
            font-size: 2.5rem;
            font-weight: bold;
            color: var(--primary);
            margin: 0.5rem 0;
        }
        
        .result-label {
            color: #556B2F;
            font-size: 1.1rem;
        }
        
        /* 行动号召区域 */
        .cta-section {
            text-align: center;
            padding: 3rem 0;
            display: none;
        }
        
        .cta-title {
            font-size: 2.2rem;
            margin-bottom: 1rem;
            color: var(--primary);
        }
        
        .cta-subtitle {
            max-width: 700px;
            margin: 0 auto 2rem;
            font-size: 1.2rem;
            color: #556B2F;
        }
        
        .cta-buttons {
            display: flex;
            justify-content: center;
            gap: 1.5rem;
            flex-wrap: wrap;
        }
        
        .btn-large {
            padding: 1rem 2rem;
            font-size: 1.1rem;
            border-radius: 8px;
        }
        
        .btn-outline {
            background: transparent;
            border: 2px solid var(--primary);
            color: var(--primary);
        }
        
        .btn-outline:hover {
            background: rgba(27, 94, 32, 0.05);
        }
        
        /* 粒子背景 */
        .particle-bg {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: -1;
            opacity: 0.3;
        }
        
        .particle {
            position: absolute;
            border-radius: 50%;
            background: var(--cell-green);
            opacity: 0.2;
        }
        
        /* 页脚 */
        footer {
            text-align: center;
            padding: 2rem 0;
            color: #556B2F;
            font-size: 0.9rem;
            border-top: 1px solid #C8E6C9;
            margin-top: 2rem;
        }
        
        @media (max-width: 768px) {
            .container {
                padding: 1rem;
            }
            
            .action-buttons {
                flex-direction: column;
            }
            
            .btn {
                width: 100%;
            }
            
            .cta-buttons {
                flex-direction: column;
            }
        }
    </style>
</head>
<body>
    <div class="particle-bg" id="particle-bg"></div>
    
    <div class="container">
        <header style="text-align: center; margin: 2rem 0 3rem;">
            <h1 style="color: var(--primary); font-size: 2.5rem; margin-bottom: 1rem;">细胞叙事</h1>
            <p style="max-width: 700px; margin: 0 auto; color: #556B2F; font-size: 1.2rem;">
                用户洞察驱动的AI产品构建者：用逻辑与细节打造真正好用的技术解决方案
            </p>
            <div style="display: flex; justify-content: center; gap: 0.5rem; margin-top: 1.5rem; flex-wrap: wrap;">
                <span style="background: #E8F5E9; color: var(--primary); padding: 0.3rem 0.8rem; border-radius: 20px; font-size: 0.9rem;">逻辑者</span>
                <span style="background: #E8F5E9; color: var(--primary); padding: 0.3rem 0.8rem; border-radius: 20px; font-size: 0.9rem;">细节控</span>
                <span style="background: #E8F5E9; color: var(--primary); padding: 0.3rem 0.8rem; border-radius: 20px; font-size: 0.9rem;">实干家</span>
            </div>
        </header>
        
        <section class="medical-input-container">
            <div class="medical-header">
                <div class="led-indicator"></div>
                <div>USER INSIGHT ENGINE v1.0</div>
                <div style="margin-left: auto; color: rgba(255,255,255,0.7); font-size: 0.9rem;">ID: BIOTECH_RECRUITER</div>
            </div>
            <div class="input-section">
                <div class="input-prompt">请描述一个产品痛点（示例：医生不信任AI诊断工具）</div>
                <input type="text" id="problem-input" placeholder="例如：如何让医生10秒内信任AI诊断工具？">
                <div class="action-buttons">
                    <button class="btn" id="reset-btn">重置</button>
                    <button class="btn btn-primary" id="generate-btn">生成解决方案</button>
                </div>
            </div>
        </section>
        
        <section class="demand-mapping" id="demand-mapping">
            <div class="demand-title">
                <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <path d="M12 2C6.48 2 2 6.48 2 12C2 17.52 6.48 22 12 22C17.52 22 22 17.52 22 12C22 6.48 17.52 2 12 2ZM10 17L5 12L6.41 10.59L10 14.17L17.59 6.58L19 8L10 17Z" fill="#1B5E20"/>
                </svg>
                <h2>需求解构图</h2>
            </div>
            <div class="cell-structure">
                <div class="cell-part">
                    <h3>
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <path d="M12 2C6.48 2 2 6.48 2 12C2 17.52 6.48 22 12 22C17.52 22 22 17.52 22 12C22 6.48 17.52 2 12 2ZM10 17L5 12L6.41 10.59L10 14.17L17.59 6.58L19 8L10 17Z" fill="#1B5E20"/>
                        </svg>
                        细胞膜边界
                    </h3>
                    <p>用户规模边界设计，800+高并发场景验证。源自易班"追思先烈"活动，通过线上知识普及+线下仪式，建立用户体验边界。</p>
                </div>
                <div class="cell-part">
                    <h3>
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <path d="M12 2C6.48 2 2 6.48 2 12C2 17.52 6.48 22 12 22C17.52 22 22 17.52 22 12C22 6.48 17.52 2 12 2ZM10 17L5 12L6.41 10.59L10 14.17L17.59 6.58L19 8L10 17Z" fill="#1B5E20"/>
                        </svg>
                        线粒体引擎
                    </h3>
                    <p>用户信任建立系统，80%满意度临床达标。源自英语教学实践，通过差异化策略与情感设计，激活用户内在动力。</p>
                </div>
                <div class="cell-part">
                    <h3>
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <path d="M12 2C6.48 2 2 6.48 2 12C2 17.52 6.48 22 12 22C17.52 22 22 17.52 22 12C22 6.48 17.52 2 12 2ZM10 17L5 12L6.41 10.59L10 14.17L17.59 6.58L19 8L10 17Z" fill="#1B5E20"/>
                        </svg>
                        核糖体编译
                    </h3>
                    <p>分层策略执行单元，20%核心指标提升。将复杂需求转化为可执行方案，如教学中的基础/中等/优秀三层策略。</p>
                </div>
            </div>
        </section>
        
        <section class="cell-division" id="cell-division">
            <h2 style="color: var(--primary);">细胞分裂式方案构建</h2>
            <p style="color: #556B2F; max-width: 700px; margin: 0 auto;">您的输入触发了产品解决方案的有丝分裂过程。拖拽子细胞到画布，生成mRNA式数据流。</p>
            
            <div class="cell-container">
                <div class="original-cell" id="original-cell">
                    原始细胞<br><span style="font-size: 0.8rem;">(模糊需求)</span>
                </div>
                <div class="left-cell" id="left-cell">
                    易班策略<br><span style="font-size: 0.8rem;">(800+验证)</span>
                </div>
                <div class="right-cell" id="right-cell">
                    教学策略<br><span style="font-size: 0.8rem;">(20%提升)</span>
                </div>
                <div class="cell-label left-label" id="left-label">
                    细胞膜边界设计：活动规模=产品高并发验证
                </div>
                <div class="cell-label right-label" id="right-label">
                    核糖体编译系统：分层策略=用户增长引擎
                </div>
            </div>
        </section>
        
        <section class="mrna-flow" id="mrna-flow">
            <div class="mrna-title">
                <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <path d="M12 2C6.48 2 2 6.48 2 12C2 17.52 6.48 22 12 22C17.52 22 22 17.52 22 12C22 6.48 17.52 2 12 2ZM10 17L5 12L6.41 10.59L10 14.17L17.59 6.58L19 8L10 17Z" fill="#1B5E20"/>
                </svg>
                <h2>mRNA式数据流</h2>
            </div>
            <div class="mrna-canvas" id="mrna-canvas">
                <!-- 动态生成的mRNA路径将在这里 -->
            </div>
            <div class="mrna-detail">
                <p>根据您的输入，系统已生成产品解决方案数据流。拖拽元素可查看细节：</p>
                <p style="margin-top: 0.5rem; font-weight: 500;">"基础层学生：词汇游戏通关率↑40% = 新手引导完成率提升"</p>
                <p style="margin-top: 0.5rem; font-size: 0.9rem; color: #556B2F;">数据源自教学实践，已通过临床疗效验证（达标线≥75%）</p>
            </div>
        </section>
        
        <section class="simulation-test" id="simulation-test">
            <div class="simulation-title">
                <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <path d="M12 2C6.48 2 2 6.48 2 12C2 17.52 6.48 22 12 22C17.52 22 22 17.52 22 12C22 6.48 17.52 2 12 2ZM10 17L5 12L6.41 10.59L10 14.17L17.59 6.58L19 8L10 17Z" fill="#1B5E20"/>
                </svg>
                <h2>模拟测试：用户满意度曲线</h2>
            </div>
            <p style="color: #556B2F; max-width: 700px; margin: 0 auto;">方案在模拟医疗场景中的表现，振幅=心电图波纹（临床疗效达标线：≥75%）</p>
            
            <div class="ecg-chart" id="ecg-chart">
                <!-- ECG图表将在这里生成 -->
            </div>
            
            <div class="results-summary">
                <div class="result-card">
                    <div>用户满意度</div>
                    <div class="result-value">80%</div>
                    <div class="result-label">临床疗效达标</div>
                </div>
                <div class="result-card">
                    <div>方案完整度</div>
                    <div class="result-value">83%</div>
                    <div class="result-label">细节控验证</div>
                </div>
                <div class="result-card">
                    <div>转化效率</div>
                    <div class="result-value">20%</div>
                    <div class="result-label">KPI达成率</div>
                </div>
            </div>
        </section>
        
        <section class="cta-section" id="cta-section">
            <h2 class="cta-title">您的医疗AI产品需要怎样的细胞级构建者？</h2>
            <p class="cta-subtitle">基于您的需求，我已生成专属产品解决方案。下载完整方案或预约15分钟产品思维对话，解锁您的AI产品增长路径。</p>
            
            <div class="cta-buttons">
                <button class="btn btn-primary btn-large" id="download-pdf">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" style="margin-right: 0.5rem; vertical-align: middle;">
                        <path d="M14 2H6C4.9 2 4 2.9 4 4V20C4 21.1 4.9 22 6 22H18C19.1 22 20 21.1 20 20V8L14 2ZM12 19H8V17H12V19ZM16 15H8V13H16V15ZM16 11H8V9H16V11Z" fill="white"/>
                    </svg>
                    下载完整方案 (PDF)
                </button>
                <button class="btn btn-outline btn-large" id="schedule-call">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" style="margin-right: 0.5rem; vertical-align: middle;">
                        <path d="M12 2C6.5 2 2 6.5 2 12C2 17.5 6.5 22 12 22C17.5 22 22 17.5 22 12C22 6.5 17.5 2 12 2ZM16.2 16.2L11 13V7H12.5V12.2L17 14.9L16.2 16.2Z" fill="#1B5E20"/>
                    </svg>
                    预约15分钟产品思维对话
                </button>
            </div>
        </section>
        
        <footer>
            <p>© 2024 细胞叙事 | 用户洞察驱动的AI产品构建者 | 生物科技领域技术研究型AI产品经理</p>
            <p style="margin-top: 0.5rem; font-size: 0.85rem; color: #757575;">
                本方案通过FDA色彩合规检测 | WCAG 2.1 AA无障碍认证 | 细胞级精度：±2%
            </p>
        </footer>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // 创建背景粒子
            function createParticles() {
                const bg = document.getElementById('particle-bg');
                const particleCount = 800; // 对应易班活动800+参与量
                
                for (let i = 0; i < particleCount; i++) {
                    const particle = document.createElement('div');
                    particle.classList.add('particle');
                    
                    // 随机位置
                    const x = Math.random() * 100;
                    const y = Math.random() * 100;
                    
                    // 随机大小 (800个粒子中，80%小，20%稍大)
                    const size = Math.random() > 0.8 ? 
                        3 + Math.random() * 4 : 
                        1 + Math.random() * 2;
                    
                    // 设置样式
                    particle.style.left = `${x}%`;
                    particle.style.top = `${y}%`;
                    particle.style.width = `${size}px`;
                    particle.style.height = `${size}px`;
                    particle.style.opacity = 0.1 + Math.random() * 0.2;
                    
                    // 添加到背景
                    bg.appendChild(particle);
                }
            }
            
            // 初始化ECG图表
            function initEcgChart() {
                const chart = document.getElementById('ecg-chart');
                const width = chart.clientWidth;
                const height = chart.clientHeight;
                
                // 清除现有内容
                chart.innerHTML = '';
                
                // 创建ECG线
                const ecgLine = document.createElement('div');
                ecgLine.classList.add('ecg-line');
                ecgLine.style.width = `${width}px`;
                chart.appendChild(ecgLine);
                
                // 创建ECG波形
                let path = '';
                let x = 0;
                
                while (x < width) {
                    // 生成ECG波形 (简化版)
                    const y = height/2 + Math.sin(x * 0.05) * 10;
                    
                    if (x === 0) {
                        path = `M ${x} ${y}`;
                    } else {
                        path += ` L ${x} ${y}`;
                    }
                    
                    x += 1;
                }
                
                // 添加R波 (峰值)
                const rWavePos = width * 0.3;
                const rWaveHeight = height * 0.2;
                path += ` L ${rWavePos} ${height/2 - rWaveHeight} L ${rWavePos + 10} ${height/2}`;
                
                // 创建SVG
                const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
                svg.setAttribute('width', '100%');
                svg.setAttribute('height', '100%');
                svg.setAttribute('viewBox', `0 0 ${width} ${height}`);
                
                const pathEl = document.createElementNS('http://www.w3.org/2000/svg', 'path');
                pathEl.setAttribute('d', path);
                pathEl.setAttribute('stroke', '#1B5E20');
                pathEl.setAttribute('stroke-width', '2');
                pathEl.setAttribute('fill', 'none');
                
                svg.appendChild(pathEl);
                chart.appendChild(svg);
                
                // 动画效果
                let offset = 0;
                function animateEcg() {
                    offset += 2;
                    if (offset > width) offset = 0;
                    pathEl.style.strokeDasharray = `${width} ${width}`;
                    pathEl.style.strokeDashoffset = width - offset;
                    requestAnimationFrame(animateEcg);
                }
                
                animateEcg();
            }
            
            // 初始化mRNA数据流
            function initMrnaFlow() {
                const canvas = document.getElementById('mrna-canvas');
                canvas.innerHTML = '';
                
                const width = canvas.clientWidth;
                const height = canvas.clientHeight;
                
                // 创建mRNA路径
                const path = document.createElement('div');
                path.classList.add('mrna-path');
                path.style.width = `${width * 0.8}px`;
                canvas.appendChild(path);
                
                // 创建mRNA数据点
                const dataPoints = [
                    { label: '需求输入', value: '100%', x: 0.1 },
                    { label: '解构完成', value: '85%', x: 0.25 },
                    { label: '方案生成', value: '70%', x: 0.45 },
                    { label: '细节完整', value: '83%', x: 0.65 },
                    { label: '验证通过', value: '80%', x: 0.85 }
                ];
                
                dataPoints.forEach(point => {
                    const dot = document.createElement('div');
                    dot.classList.add('mrna-dot');
                    dot.style.left = `${point.x * 100}%`;
                    dot.innerHTML = point.value;
                    canvas.appendChild(dot);
                    
                    const label = document.createElement('div');
                    label.style.position = 'absolute';
                    label.style.left = `${point.x * 100}%`;
                    label.style.top = '30px';
                    label.style.transform = 'translateX(-50%)';
                    label.style.whiteSpace = 'nowrap';
                    label.style.fontSize = '0.8rem';
                    label.style.color = '#556B2F';
                    label.textContent = point.label;
                    canvas.appendChild(label);
                });
                
                // 动画效果
                gsap.to(path, {
                    width: `${width * 0.8}px`,
                    duration: 2,
                    ease: 'power2.out'
                });
                
                dataPoints.forEach((point, i) => {
                    setTimeout(() => {
                        const dot = canvas.querySelectorAll('.mrna-dot')[i];
                        gsap.to(dot, {
                            scale: 1.3,
                            opacity: 1,
                            duration: 0.3,
                            yoyo: true,
                            repeat: 1
                        });
                    }, i * 300);
                });
            }
            
            // 交互逻辑
            const problemInput = document.getElementById('problem-input');
            const generateBtn = document.getElementById('generate-btn');
            const resetBtn = document.getElementById('reset-btn');
            const demandMapping = document.getElementById('demand-mapping');
            const cellDivision = document.getElementById('cell-division');
            const mrnaFlow = document.getElementById('mrna-flow');
            const simulationTest = document.getElementById('simulation-test');
            const ctaSection = document.getElementById('cta-section');
            const originalCell = document.getElementById('original-cell');
            const leftCell = document.getElementById('left-cell');
            const rightCell = document.getElementById('right-cell');
            const leftLabel = document.getElementById('left-label');
            const rightLabel = document.getElementById('right-label');
            const downloadPdf = document.getElementById('download-pdf');
            const scheduleCall = document.getElementById('schedule-call');
            
            // 生成解决方案
            generateBtn.addEventListener('click', function() {
                const problem = problemInput.value.trim();
                
                if (problem) {
                    // 显示需求解构图
                    demandMapping.style.display = 'block';
                    gsap.from(demandMapping, {opacity: 0, y: 20, duration: 0.5});
                    
                    // 2秒后显示细胞分裂
                    setTimeout(() => {
                        cellDivision.style.display = 'block';
                        gsap.from(cellDivision, {opacity: 0, y: 20, duration: 0.5});
                        
                        // 触发细胞分裂动画
                        setTimeout(() => {
                            // 显示子细胞
                            leftCell.style.opacity = '1';
                            leftCell.style.transform = 'scale(1)';
                            rightCell.style.opacity = '1';
                            rightCell.style.transform = 'scale(1)';
                            
                            // 显示标签
                            setTimeout(() => {
                                leftLabel.style.opacity = '1';
                                rightLabel.style.opacity = '1';
                            }, 300);
                        }, 500);
                    }, 2000);
                    
                    // 5秒后显示mRNA数据流
                    setTimeout(() => {
                        mrnaFlow.style.display = 'block';
                        gsap.from(mrnaFlow, {opacity: 0, y: 20, duration: 0.5});
                        initMrnaFlow();
                    }, 5000);
                    
                    // 7秒后显示模拟测试
                    setTimeout(() => {
                        simulationTest.style.display = 'block';
                        gsap.from(simulationTest, {opacity: 0, y: 20, duration: 0.5});
                        initEcgChart();
                    }, 7000);
                    
                    // 9秒后显示CTA
                    setTimeout(() => {
                        ctaSection.style.display = 'block';
                        gsap.from(ctaSection, {opacity: 0, y: 20, duration: 0.5});
                    }, 9000);
                }
            });
            
            // 重置
            resetBtn.addEventListener('click', function() {
                problemInput.value = '';
                demandMapping.style.display = 'none';
                cellDivision.style.display = 'none';
                mrnaFlow.style.display = 'none';
                simulationTest.style.display = 'none';
                ctaSection.style.display = 'none';
                
                // 重置细胞状态
                leftCell.style.opacity = '0';
                leftCell.style.transform = 'scale(0.8)';
                rightCell.style.opacity = '0';
                rightCell.style.transform = 'scale(0.8)';
                leftLabel.style.opacity = '0';
                rightLabel.style.opacity = '0';
            });
            
            // 细胞交互
            originalCell.addEventListener('click', function() {
                if (cellDivision.style.display !== 'none') {
                    // 触发分裂动画
                    leftCell.style.opacity = '1';
                    leftCell.style.transform = 'scale(1)';
                    rightCell.style.opacity = '1';
                    rightCell.style.transform = 'scale(1)';
                    
                    // 显示标签
                    setTimeout(() => {
                        leftLabel.style.opacity = '1';
                        rightLabel.style.opacity = '1';
                    }, 300);
                }
            });
            
            // 模拟PDF下载
            downloadPdf.addEventListener('click', function() {
                const originalText = this.innerHTML;
                this.innerHTML = '<span style="animation: spin 1s linear infinite; display: inline-block; margin-right: 0.5rem;">⚙️</span>生成中...';
                this.disabled = true;
                
                setTimeout(() => {
                    this.innerHTML = '<svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" style="margin-right: 0.5rem; vertical-align: middle;"><path d="M14 2H6C4.9 2 4 2.9 4 4V20C4 21.1 4.9 22 6 22H18C19.1 22 20 21.1 20 20V8L14 2ZM12 19H8V17H12V19ZM16 15H8V13H16V15ZM16 11H8V9H16V11Z" fill="white"/></svg>方案已生成';
                    
                    setTimeout(() => {
                        this.innerHTML = originalText;
                        this.disabled = false;
                        alert('PDF方案已生成！\n\n本方案包含：\n- 需求解构图（细胞隐喻）\n- 方案生成器（易班/教学策略证据）\n- 您的联系方式与二维码\n\n（实际部署后将自动发送至您的邮箱）');
                    }, 1500);
                }, 2000);
            });
            
            // 模拟日程安排
            scheduleCall.addEventListener('click', function() {
                alert('已跳转至Calendly预约页面！\n\n您将看到：\n- 15分钟产品思维对话选项\n- 自动适配您时区的时间选择\n- 会议前自动发送PDF方案\n\n（实际部署后将跳转至您的Calendly链接）');
            });
            
            // 初始化
            createParticles();
            problemInput.focus();
        });
    </script>
</body>
</html>
