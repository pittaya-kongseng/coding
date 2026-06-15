import React, { useState, useEffect, useCallback } from 'react';
import { 
  ChevronRight, 
  ChevronLeft, 
  ChevronUp, 
  ChevronDown, 
  Play, 
  RotateCcw,  
  Map, 
  BookOpen, 
  Settings,
  X
} from 'lucide-react';

// --- Level Generator (50 Levels) ---
const generateLevels = () => {
  const levels = [];
  
  // 1-5: มือใหม่ (ง่ายมาก)
  levels.push({ id: 1, grid: { w: 3, h: 1 }, start: {x: 0, y: 0}, coins: [{x: 2, y: 0}], walls: [] });
  levels.push({ id: 2, grid: { w: 4, h: 1 }, start: {x: 0, y: 0}, coins: [{x: 3, y: 0}], walls: [] });
  levels.push({ id: 3, grid: { w: 3, h: 3 }, start: {x: 0, y: 0}, coins: [{x: 2, y: 2}], walls: [] });
  levels.push({ id: 4, grid: { w: 3, h: 3 }, start: {x: 0, y: 0}, coins: [{x: 2, y: 2}], walls: [{x: 1, y: 0}, {x: 1, y: 1}] });
  levels.push({ id: 5, grid: { w: 4, h: 4 }, start: {x: 0, y: 3}, coins: [{x: 3, y: 0}], walls: [{x: 1, y: 3}, {x: 2, y: 1}, {x: 3, y: 2}] });

  // 6-50: สร้างอัตโนมัติ ไล่ระดับความยาก
  for (let i = 6; i <= 50; i++) {
    // ขนาดตารางค่อยๆ ใหญ่ขึ้น (สูงสุด 8x8)
    const size = Math.min(8, 3 + Math.floor(i / 10));
    const w = size;
    const h = size;
    
    const start = { x: 0, y: h - 1 };
    const coins = [{ x: w - 1, y: 0 }];
    const walls = [];

    // เพิ่มกำแพงตามระดับด่าน (ด่านยิ่งสูง กำแพงยิ่งเยอะ)
    const numWalls = Math.floor(i / 5);
    for (let j = 0; j < numWalls; j++) {
      const wx = Math.floor(Math.random() * w);
      const wy = Math.floor(Math.random() * h);
      // ห้ามทับจุดเกิด และจุดเหรียญ
      if ((wx !== start.x || wy !== start.y) && (wx !== coins[0].x || wy !== coins[0].y)) {
        // เช็คซ้ำ
        if (!walls.some(wall => wall.x === wx && wall.y === wy)) {
          walls.push({ x: wx, y: wy });
        }
      }
    }

    levels.push({ id: i, grid: { w, h }, start, coins, walls });
  }
  return levels;
};

const ALL_LEVELS = generateLevels();

export default function App() {
  const [currentLevelIndex, setCurrentLevelIndex] = useState(0);
  const level = ALL_LEVELS[currentLevelIndex];
  
  const [codeBlocks, setCodeBlocks] = useState([]);
  const [robotPos, setRobotPos] = useState({ ...level.start });
  const [coinsLeft, setCoinsLeft] = useState([...level.coins]);
  const [coinsCollected, setCoinsCollected] = useState(0);
  
  const [gameStatus, setGameStatus] = useState('idle'); // idle, running, success, fail
  const [activeBlockIndex, setActiveBlockIndex] = useState(-1);

  // Reset level state when level changes or reset is clicked
  const resetLevel = useCallback(() => {
    setRobotPos({ ...level.start });
    setCoinsLeft([...level.coins]);
    setCoinsCollected(0);
    setGameStatus('idle');
    setActiveBlockIndex(-1);
  }, [level]);

  useEffect(() => {
    resetLevel();
    setCodeBlocks([]); // Clear code only on level change or manual full reset
  }, [currentLevelIndex, resetLevel]);

  const addBlock = (direction) => {
    if (gameStatus === 'running') return;
    if (codeBlocks.length >= 20) return; // Limit max blocks
    setCodeBlocks([...codeBlocks, direction]);
  };

  const removeBlock = (index) => {
    if (gameStatus === 'running') return;
    const newBlocks = [...codeBlocks];
    newBlocks.splice(index, 1);
    setCodeBlocks(newBlocks);
  };

  const executeCode = async () => {
    if (codeBlocks.length === 0 || gameStatus === 'running') return;
    
    // Reset positions before running
    resetLevel();
    setGameStatus('running');
    
    let currentX = level.start.x;
    let currentY = level.start.y;
    let currentCoins = [...level.coins];
    let collected = 0;

    // Small delay before starting
    await new Promise(r => setTimeout(r, 300));

    for (let i = 0; i < codeBlocks.length; i++) {
      setActiveBlockIndex(i);
      const block = codeBlocks[i];
      
      if (block === 'RIGHT') currentX++;
      else if (block === 'LEFT') currentX--;
      else if (block === 'DOWN') currentY++;
      else if (block === 'UP') currentY--;

      // Check boundaries
      if (currentX < 0 || currentX >= level.grid.w || currentY < 0 || currentY >= level.grid.h) {
        setGameStatus('fail');
        setActiveBlockIndex(-1);
        return;
      }

      // Check walls
      if (level.walls.some(w => w.x === currentX && w.y === currentY)) {
        setGameStatus('fail');
        setActiveBlockIndex(-1);
        return;
      }

      setRobotPos({ x: currentX, y: currentY });

      // Check coin collection
      const coinIndex = currentCoins.findIndex(c => c.x === currentX && c.y === currentY);
      if (coinIndex > -1) {
        collected++;
        currentCoins.splice(coinIndex, 1);
        setCoinsLeft(currentCoins);
        setCoinsCollected(collected);
      }

      // Wait for animation
      await new Promise(r => setTimeout(r, 500));
    }

    setActiveBlockIndex(-1);
    
    // Check win condition
    if (collected >= level.coins.length) {
      setGameStatus('success');
    } else {
      setGameStatus('fail');
    }
  };

  const nextLevel = () => {
    if (currentLevelIndex < ALL_LEVELS.length - 1) {
      setCurrentLevelIndex(prev => prev + 1);
    }
  };

  // Render Helpers
  const renderIcon = (type) => {
    switch (type) {
      case 'LEFT': return <ChevronLeft size={28} className="text-white" strokeWidth={3} />;
      case 'RIGHT': return <ChevronRight size={28} className="text-white" strokeWidth={3} />;
      case 'UP': return <ChevronUp size={28} className="text-white" strokeWidth={3} />;
      case 'DOWN': return <ChevronDown size={28} className="text-white" strokeWidth={3} />;
      default: return null;
    }
  };

  return (
    <div className="flex flex-col h-screen w-full bg-[#0d1f3d] font-sans overflow-hidden">
      
      {/* Top Navigation Bar */}
      <div className="flex items-center justify-between px-4 py-3 bg-[#112240] text-white shadow-md z-10">
        <div className="flex items-center space-x-4">
          <button className="flex items-center text-gray-300 hover:text-white transition-colors">
            <ChevronLeft size={24} />
            <span className="ml-1 font-semibold tracking-wide">หลักสูตร</span>
          </button>
          <div className="h-6 w-px bg-slate-600 mx-2"></div>
          <span className="font-bold text-blue-400 tracking-wider">KMS_KidsCanCode</span>
        </div>
        <div className="flex items-center space-x-6 text-gray-400">
          <Map size={20} className="hover:text-white cursor-pointer transition-colors" />
          <BookOpen size={20} className="hover:text-white cursor-pointer transition-colors" />
          <Settings size={20} className="hover:text-white cursor-pointer transition-colors" />
        </div>
      </div>

      {/* Main Content Area */}
      <div className="flex flex-1 overflow-hidden">
        
        {/* LEFT PANEL (Code Editor) */}
        <div className="w-[40%] min-w-[320px] bg-[#0A192F] flex flex-col border-r border-[#1e365d] relative z-0">
          
          {/* Workspace Area */}
          <div className="flex-1 p-6 flex flex-col gap-4 overflow-y-auto">
            <div className="min-h-[250px] border-2 border-blue-800 rounded-lg bg-[#0d213f] p-4 shadow-inner flex flex-wrap content-start gap-2">
              {/* Start Block */}
              <div className="w-14 h-14 bg-emerald-500 rounded-lg flex items-center justify-center shadow-lg shadow-emerald-500/20 z-10 shrink-0">
                <Play size={24} fill="white" className="text-white ml-1" />
              </div>
              
              {/* Added Code Blocks */}
              {codeBlocks.map((block, index) => (
                <div 
                  key={index} 
                  onClick={() => removeBlock(index)}
                  className={`
                    w-14 h-14 bg-[#1E88E5] rounded-lg flex items-center justify-center cursor-pointer
                    hover:bg-blue-400 transition-all shrink-0 relative group shadow-md
                    ${activeBlockIndex === index ? 'ring-4 ring-yellow-400 scale-110 z-20' : 'z-10'}
                  `}
                >
                  {renderIcon(block)}
                  {/* Tooltip on hover to delete */}
                  <div className="absolute -top-2 -right-2 bg-red-500 rounded-full p-0.5 opacity-0 group-hover:opacity-100 transition-opacity">
                    <X size={12} className="text-white" />
                  </div>
                </div>
              ))}

              {/* Placeholder for next block */}
              {codeBlocks.length < 20 && (
                <div className="w-14 h-14 border-2 border-dashed border-blue-700/50 rounded-lg flex items-center justify-center shrink-0">
                </div>
              )}
            </div>

            {/* Input Palette */}
            <div className="mt-4">
              <h3 className="text-white font-bold mb-4">เลือกคำสั่ง</h3>
              <div className="flex gap-4">
                {['LEFT', 'UP', 'DOWN', 'RIGHT'].map((dir) => (
                  <button 
                    key={dir}
                    onClick={() => addBlock(dir)}
                    disabled={gameStatus === 'running'}
                    className={`
                      w-16 h-16 bg-[#1E88E5] rounded-xl flex items-center justify-center
                      shadow-[0_4px_0_#1565C0] active:translate-y-1 active:shadow-none
                      hover:bg-blue-400 transition-all
                      ${gameStatus === 'running' ? 'opacity-50 cursor-not-allowed' : ''}
                    `}
                  >
                    {renderIcon(dir)}
                  </button>
                ))}
              </div>
            </div>
          </div>

          {/* Action Buttons */}
          <div className="p-6 flex gap-4 bg-[#0A192F]">
            <button 
              onClick={() => { resetLevel(); setCodeBlocks([]); }}
              className="flex-1 py-4 bg-white text-slate-700 font-bold rounded-xl flex items-center justify-center gap-2 hover:bg-gray-100 active:bg-gray-200 transition-colors shadow-sm"
            >
              <RotateCcw size={20} />
              เริ่มใหม่
            </button>
            <button 
              onClick={executeCode}
              disabled={gameStatus === 'running' || codeBlocks.length === 0}
              className={`
                flex-1 py-4 font-bold rounded-xl flex items-center justify-center gap-2 transition-all shadow-sm
                ${(gameStatus === 'running' || codeBlocks.length === 0) 
                  ? 'bg-gray-400 text-gray-200 cursor-not-allowed' 
                  : 'bg-white text-slate-800 hover:bg-gray-100 active:bg-gray-200'}
              `}
            >
              <Play size={20} className={codeBlocks.length > 0 && gameStatus !== 'running' ? 'text-green-500' : ''} fill={codeBlocks.length > 0 && gameStatus !== 'running' ? 'currentColor' : 'none'} />
              เล่น
            </button>
          </div>
        </div>

        {/* RIGHT PANEL (Game Canvas) */}
        <div className="w-[60%] bg-[#112240] relative flex flex-col">
          
          {/* Header Info */}
          <div className="flex justify-between items-center p-6 text-white font-bold">
            <div className="flex items-center gap-2 bg-slate-800/50 px-4 py-2 rounded-full border border-slate-700">
              <div className="w-6 h-6 bg-gradient-to-b from-yellow-300 to-yellow-500 rounded-full border-2 border-yellow-600 flex items-center justify-center shadow-[0_0_8px_rgba(253,224,71,0.5)]">
                <span className="text-yellow-800 text-xs font-black">$</span>
              </div>
              <span>{coinsCollected}/{level.coins.length}</span>
            </div>
            <div className="text-lg tracking-widest text-slate-300">
              ด่าน {currentLevelIndex + 1} - 50
            </div>
          </div>

          {/* Game Grid Container */}
          <div className="flex-1 flex items-center justify-center p-8">
            <div 
              className="bg-[#0A192F] rounded-xl shadow-2xl p-2 relative"
              style={{
                display: 'grid',
                gridTemplateColumns: `repeat(${level.grid.w}, minmax(0, 1fr))`,
                gap: '8px',
                // Make cells square and responsive
                width: 'min(100%, 600px)',
                aspectRatio: `${level.grid.w} / ${level.grid.h}`
              }}
            >
              {/* Generate Grid Cells */}
              {Array.from({ length: level.grid.h * level.grid.w }).map((_, index) => {
                const x = index % level.grid.w;
                const y = Math.floor(index / level.grid.w);
                const isWall = level.walls.some(w => w.x === x && w.y === y);
                
                return (
                  <div 
                    key={index}
                    className={`
                      rounded-lg flex items-center justify-center relative
                      ${isWall ? 'bg-slate-700 border-b-4 border-slate-900' : 'bg-[#e2e8f0] shadow-inner'}
                    `}
                  >
                    {/* Render Coin if exists at this cell */}
                    {coinsLeft.some(c => c.x === x && c.y === y) && (
                      <div className="animate-bounce z-10">
                        <div className="w-8 h-8 md:w-10 md:h-10 bg-gradient-to-b from-yellow-300 to-yellow-500 rounded-full border-[3px] border-yellow-600 flex items-center justify-center drop-shadow-lg shadow-[0_0_15px_rgba(253,224,71,0.6)] relative">
                          <div className="absolute inset-[2px] border border-yellow-600/40 rounded-full"></div>
                          <span className="text-yellow-800 text-lg md:text-xl font-black drop-shadow-sm">$</span>
                        </div>
                      </div>
                    )}

                    {/* Render Wall decoration */}
                    {isWall && (
                      <div className="w-full h-full bg-[repeating-linear-gradient(45deg,transparent,transparent_10px,rgba(0,0,0,0.1)_10px,rgba(0,0,0,0.1)_20px)]"></div>
                    )}
                  </div>
                );
              })}

              {/* Render Robot (Absolutely positioned over the grid) */}
              <div 
                className="absolute z-10 flex items-center justify-center transition-all duration-500 ease-in-out"
                style={{
                  width: `calc(100% / ${level.grid.w})`,
                  height: `calc(100% / ${level.grid.h})`,
                  left: `${(robotPos.x / level.grid.w) * 100}%`,
                  top: `${(robotPos.y / level.grid.h) * 100}%`,
                }}
              >
                <div className={`
                  text-4xl md:text-6xl filter drop-shadow-lg
                  ${gameStatus === 'fail' ? 'grayscale opacity-80 rotate-180 transition-all' : ''}
                  ${gameStatus === 'success' ? 'animate-bounce' : ''}
                `}>
                  🤖
                </div>
              </div>

            </div>
          </div>

          {/* Overlays (Success / Fail) */}
          {gameStatus === 'success' && (
            <div className="absolute inset-0 bg-slate-900/80 backdrop-blur-sm flex flex-col items-center justify-center z-50">
              <div className="text-6xl mb-4">🌟</div>
              <h2 className="text-4xl font-bold text-white mb-6">ยอดเยี่ยมมาก!</h2>
              {currentLevelIndex < ALL_LEVELS.length - 1 ? (
                <button 
                  onClick={nextLevel}
                  className="bg-green-500 hover:bg-green-600 text-white text-xl font-bold py-4 px-12 rounded-full shadow-lg shadow-green-500/30 transition-transform hover:scale-105 active:scale-95"
                >
                  ด่านต่อไป
                </button>
              ) : (
                <div className="text-2xl text-yellow-400 font-bold">คุณเคลียร์ครบ 50 ด่านแล้ว! 🎉</div>
              )}
            </div>
          )}

          {gameStatus === 'fail' && (
            <div className="absolute inset-0 bg-red-900/40 backdrop-blur-sm flex flex-col items-center justify-center z-50">
              <h2 className="text-4xl font-bold text-white mb-6 drop-shadow-lg">อุ๊ปส์! ลองใหม่อีกครั้งนะ</h2>
              <button 
                onClick={resetLevel}
                className="bg-white text-red-600 text-xl font-bold py-4 px-12 rounded-full shadow-lg transition-transform hover:scale-105 active:scale-95 flex items-center gap-2"
              >
                <RotateCcw size={24} /> เริ่มด่านนี้ใหม่
              </button>
            </div>
          )}

          {/* Footer Decoration & Buddy */}
          <div className="h-16 w-full bg-[#0a162b] border-t border-slate-700/50 absolute bottom-0 flex items-center px-4 overflow-hidden">
             {/* Decorative tech/pipe background elements */}
             <div className="flex gap-8 opacity-20">
               <div className="w-12 h-8 border-2 border-cyan-500 rounded flex items-center justify-center"><div className="w-8 h-2 bg-cyan-500"></div></div>
               <div className="w-8 h-8 rounded-full border-2 border-purple-500 flex items-center justify-center"><div className="w-2 h-2 bg-purple-500 rounded-full"></div></div>
               <div className="w-24 h-4 border-y-2 border-blue-500"></div>
             </div>
          </div>

          {/* Buddy Icon */}
          <div className="absolute bottom-6 right-6 z-20 flex flex-col items-center">
            <div className="w-16 h-16 bg-white rounded-full border-4 border-slate-800 shadow-xl flex items-center justify-center text-3xl overflow-hidden cursor-pointer hover:scale-105 transition-transform">
              <div className="bg-[#1e3a8a] w-full h-full flex items-center justify-center">
                 <div className="w-10 h-6 bg-yellow-400 rounded-full relative">
                    <div className="absolute top-1 left-2 w-2 h-2 bg-slate-900 rounded-full"></div>
                    <div className="absolute top-1 right-2 w-2 h-2 bg-slate-900 rounded-full"></div>
                 </div>
              </div>
            </div>
            <span className="text-white text-sm font-bold mt-2 bg-slate-800 px-3 py-1 rounded-full border border-slate-700">Buddy</span>
          </div>

        </div>
      </div>
    </div>
  );
}
