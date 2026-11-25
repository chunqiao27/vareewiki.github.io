# vareewiki.github.io
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>วารีมากิ (Varee Maki) — แบบจำลอง 3 มิติ 5 ขั้นตอน</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+Thai:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'Noto Sans Thai', 'Inter', sans-serif;
    }
    #canvas-container canvas {
      display: block;
      width: 100%;
      height: 100%;
      cursor: grab;
      touch-action: none;
    }
    #canvas-container canvas:active {
      cursor: grabbing;
    }
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-thumb {
      background: rgba(100, 116, 139, 0.4);
      border-radius: 9999px;
    }
  </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col overflow-x-hidden select-none">
  <!-- Top Header -->
  <header class="bg-slate-900/90 border-b border-slate-800 px-4 py-3 flex flex-wrap items-center justify-between gap-3">
    <div class="flex items-center gap-3 min-w-0">
      <div class="w-10 h-10 rounded-xl bg-teal-500/20 border border-teal-400/40 flex items-center justify-center text-teal-300 font-bold text-lg shrink-0">
        VM
      </div>
      <div class="min-w-0">
        <h1 class="text-base sm:text-lg font-bold text-white truncate">
          วารีมากิ (Varee Maki) — แบบจำลอง 3 มิติ
        </h1>
        <p class="text-xs text-slate-400 truncate">
          เตียงพยุงตัวพร้อมระบบเปลี่ยนแผ่นรองแบบม้วนถอดซัก สำหรับสุนัขขนาดกลางถึงใหญ่
        </p>
      </div>
    </div>

    <!-- Camera Presets -->
    <div class="flex items-center gap-1.5 flex-wrap" role="group" aria-label="มุมกล้อง">
      <span class="text-xs text-slate-400 mr-1 hidden md:inline">มุมกล้อง:</span>
      <button data-cam="overview" class="cam-btn px-3 py-1.5 rounded-lg text-xs font-medium bg-teal-600 text-white border border-teal-400/50 transition">
        ภาพรวม
      </button>
      <button data-cam="front" class="cam-btn px-3 py-1.5 rounded-lg text-xs font-medium bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 transition">
        ด้านหน้า (ช่องออกแผ่นเปื้อน)
      </button>
      <button data-cam="back" class="cam-btn px-3 py-1.5 rounded-lg text-xs font-medium bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 transition">
        ด้านหลัง (ตลับผ้าสะอาด)
      </button>
      <button data-cam="side" class="cam-btn px-3 py-1.5 rounded-lg text-xs font-medium bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 transition">
        ด้านข้าง (กลไกยกและมือหมุน)
      </button>
    </div>
  </header>

  <!-- Main Responsive Layout -->
  <main class="flex-1 grid grid-cols-1 lg:grid-cols-12 gap-4 p-3 sm:p-4 max-w-[1600px] w-full mx-auto">
    <!-- 3D Viewport & Playback Bar -->
    <section class="lg:col-span-8 flex flex-col gap-3 min-h-[380px] sm:min-h-[460px]">
      <div class="relative flex-1 rounded-2xl overflow-hidden border border-slate-800 bg-slate-900 shadow-2xl min-h-[340px]">
        <div id="canvas-container" class="w-full h-full"></div>

        <!-- Floating Status Pill -->
        <div class="absolute top-3 left-3 right-3 flex items-start justify-between gap-2 pointer-events-none">
          <div id="status-badge" class="pointer-events-auto bg-slate-950/80 backdrop-blur-md border border-slate-700/80 rounded-xl px-3.5 py-2 shadow-lg max-w-md">
            <div class="flex items-center gap-2 text-xs font-semibold text-teal-300">
              <span id="step-indicator-num">ขั้นตอนที่ 1 / 5</span>
              <span class="text-slate-600">•</span>
              <span id="lock-status-text" class="text-emerald-400">สถานะ: พักนอนแนบแผ่นรองและฐานนุ่ม</span>
            </div>
            <div id="step-indicator-title" class="text-sm font-bold text-white mt-0.5">
              ขั้นตอนที่ 1: พักนอน (Resting State)
            </div>
          </div>

          <button id="reset-view-btn" class="pointer-events-auto bg-slate-950/80 hover:bg-slate-800 backdrop-blur-md border border-slate-700 text-slate-200 text-xs font-medium px-3 py-2 rounded-xl shadow-lg transition">
            รีเซ็ตมุมกล้อง
          </button>
        </div>

        <!-- Bottom Hint -->
        <div class="absolute bottom-3 left-3 bg-slate-950/75 backdrop-blur-sm border border-slate-800 rounded-lg px-3 py-1.5 text-[11px] text-slate-300 pointer-events-none">
          ลากเมาส์หรือใช้นิ้วหมุนดูรอบทิศทาง • เลื่อนล้อหรือบีบนิ้วเพื่อซูม • แสดงเฉพาะชิ้นส่วนและกลไกหลักของระบบ
        </div>
      </div>

      <!-- Playback Controls -->
      <div class="bg-slate-900 border border-slate-800 rounded-2xl p-3 flex flex-wrap items-center justify-between gap-3">
        <div class="flex items-center gap-2">
          <button id="prev-btn" class="px-3.5 py-2 rounded-xl text-xs sm:text-sm font-semibold bg-slate-800 hover:bg-slate-700 disabled:opacity-40 text-white border border-slate-700 transition">
            ← ก่อนหน้า
          </button>
          <button id="play-pause-btn" class="px-4 py-2 rounded-xl text-xs sm:text-sm font-semibold bg-teal-500 hover:bg-teal-400 text-slate-950 shadow-md transition flex items-center gap-1.5">
            <span id="play-pause-icon">⏸</span>
            <span id="play-pause-label">หยุดชั่วคราว</span>
          </button>
          <button id="next-btn" class="px-3.5 py-2 rounded-xl text-xs sm:text-sm font-semibold bg-slate-800 hover:bg-slate-700 disabled:opacity-40 text-white border border-slate-700 transition">
            ถัดไป →
          </button>
        </div>

        <!-- Step Progress Bar -->
        <div class="flex items-center gap-2 flex-1 min-w-[180px] max-w-sm">
          <span class="text-xs text-slate-400 whitespace-nowrap">ความคืบหน้า</span>
          <div class="w-full h-2.5 bg-slate-800 rounded-full overflow-hidden border border-slate-700/60">
            <div id="progress-bar" class="h-full bg-gradient-to-r from-teal-400 to-emerald-400 rounded-full transition-all duration-300" style="width: 20%;"></div>
          </div>
          <span id="progress-pct" class="text-xs font-semibold text-teal-300 w-10 text-right">20%</span>
        </div>
      </div>
    </section>

    <!-- Right Panel: 5-Step Workflow & Technical Details -->
    <aside class="lg:col-span-4 flex flex-col gap-3">
      <!-- Active Step Detail Card -->
      <div class="bg-slate-900 border border-slate-800 rounded-2xl p-4 shadow-xl">
        <div class="flex items-center justify-between gap-2 mb-2">
          <span id="detail-badge" class="text-xs font-bold uppercase tracking-wider px-2.5 py-1 rounded-full bg-teal-500/15 text-teal-300 border border-teal-400/30">
            ขั้นตอนที่ 1
          </span>
          <span id="detail-crank-tag" class="text-xs text-slate-400">
            กลไก: ตำแหน่งพักนอน (ยกพ้นพื้น)
          </span>
        </div>
        <h2 id="detail-title" class="text-base sm:text-lg font-bold text-white mb-2">
          ขั้นตอนที่ 1: พักนอน (Resting State)
        </h2>
        <p id="detail-desc" class="text-xs sm:text-sm text-slate-300 leading-relaxed">
          เปลตาข่ายแนบกับแผ่นรองผ้าฝ้ายและฐานนุ่มที่อยู่ห่างจากพื้นอย่างมั่นคง สุนัขขนาดกลางถึงใหญ่พักผ่อนอย่างสบาย แนวเชื่อมต่อซิปของแผ่นรองอยู่พ้นบริเวณรองรับน้ำหนักตัวสัตว์
        </p>
      </div>

      <!-- Interactive Step List (1 to 5) -->
      <div class="bg-slate-900 border border-slate-800 rounded-2xl p-3 flex-1 flex flex-col gap-2">
        <div class="px-1 pt-0.5 pb-1 flex items-center justify-between">
          <span class="text-xs font-semibold text-slate-300">เลือกขั้นตอนการทำงาน (คลิกเพื่อดู)</span>
          <span class="text-[11px] text-emerald-400 font-medium">รวมขั้นตอนที่ 4 เพิ่มผ้าใหม่</span>
        </div>

        <div id="step-list" class="flex flex-col gap-2 overflow-y-auto pr-0.5"></div>
      </div>

      <!-- System Legend -->
      <div class="bg-slate-900/80 border border-slate-800 rounded-2xl p-3.5 text-xs text-slate-300">
        <div class="font-semibold text-slate-200 mb-2">องค์ประกอบหลักในแบบจำลอง</div>
        <div class="grid grid-cols-2 gap-2">
          <div class="flex items-center gap-2">
            <span class="w-3 h-3 rounded-sm bg-sky-400 inline-block shrink-0"></span>
            <span class="truncate">เปลตาข่ายพยุงตัว</span>
          </div>
          <div class="flex items-center gap-2">
            <span class="w-3 h-3 rounded-sm bg-amber-100 inline-block shrink-0"></span>
            <span class="truncate">แผ่นรองผ้าฝ้าย (ม้วน/ซิป)</span>
          </div>
          <div class="flex items-center gap-2">
            <span class="w-3 h-3 rounded-sm bg-emerald-500 inline-block shrink-0"></span>
            <span class="truncate">แผ่นรองใหม่นำเข้าใช้</span>
          </div>
          <div class="flex items-center gap-2">
            <span class="w-3 h-3 rounded-sm bg-amber-600 inline-block shrink-0"></span>
            <span class="truncate">แผ่นเปื้อนออกด้านหน้า</span>
          </div>
        </div>
      </div>
    </aside>
  </main>

  <script>
    (function () {
      'use strict';

      // ==========================================
      // 1. STEP DEFINITIONS (5 STEPS)
      // ==========================================
      const STEPS = [
        {
          id: 1,
          shortTitle: 'พักนอน',
          title: 'ขั้นตอนที่ 1: พักนอน (Resting State)',
          crankTag: 'กลไก: ตำแหน่งพักนอน (ยกพ้นพื้น)',
          lockText: 'สถานะ: พักนอนแนบแผ่นรองและฐานนุ่ม',
          desc: 'เปลตาข่ายแนบกับแผ่นรองผ้าฝ้ายและฐานนุ่มที่อยู่ห่างจากพื้นอย่างมั่นคง สุนัขขนาดกลางถึงใหญ่พักผ่อนอย่างสบาย แนวเชื่อมต่อซิปของแผ่นรองอยู่พ้นบริเวณรองรับน้ำหนักตัวสัตว์',
          cam: 'overview'
        },
        {
          id: 2,
          shortTitle: 'เตรียมและยกแผ่นรองลง',
          title: 'ขั้นตอนที่ 2: เตรียมและยกแผ่นรองลง (Clear Solid Waste & Lower Underpad Assembly)',
          crankTag: 'กลไก: หมุนมือหมุนลดชุดแผ่นรองลง + ล็อกกรอบ',
          lockText: 'สถานะ: ยกแผ่นรองลงและล็อกกรอบแล้ว',
          desc: 'เก็บสิ่งปฏิกูลที่เป็นก้อนออก จากนั้นหมุนมือหมุนควบคุมเพื่อยกชุดแผ่นรองและฐานนุ่มลดระดับลงด้านล่างให้พ้นตัวสุนัขที่เปลตาข่ายพยุงไว้ แล้วล็อกกรอบให้มั่นคงก่อนเลื่อนผ้า',
          cam: 'side'
        },
        {
          id: 3,
          shortTitle: 'หมุนเลื่อนและปลดแผ่นเปื้อน',
          title: 'ขั้นตอนที่ 3: หมุนเลื่อนและปลดแผ่นเปื้อน (Feed Pad & Detach Used Pad)',
          crankTag: 'กลไก: หมุนเลื่อนแผ่นรอง + ปลดซิปด้านหน้า',
          lockText: 'สถานะ: เลื่อนแผ่นเปื้อนออกและปลดซิป',
          desc: 'หมุนมือหมุนป้อนแผ่นรอง (Pad feed crank) เพื่อเลื่อนแผ่นรองไปข้างหน้า แผ่นเปื้อนเคลื่อนออกทางช่องด้านหน้า (Used pad exit) และปลดซิปแยกออกเพื่อนำไปซัก โดยไม่สะสมเป็นม้วนเปื้อนในเครื่อง',
          cam: 'front'
        },
        {
          id: 4,
          shortTitle: 'เอาผ้าใหม่มาใช้และเชื่อมต่อ',
          title: 'ขั้นตอนที่ 4: เอาผ้าใหม่มาใช้และเชื่อมต่อ (Bring New Pad Into Use & Zip Connect)',
          crankTag: 'กลไก: นำแผ่นรองใหม่เข้าตลับ + เชื่อมซิป',
          lockText: 'สถานะ: นำผ้าใหม่เข้าสู่ระบบและเชื่อมต่อแล้ว',
          desc: 'ผู้ดูแลนำแผ่นรองผ้าฝ้ายผืนใหม่ (หรือผืนที่ซักสะอาดแล้ว) วางบรรจุลงในตลับเก็บแผ่นสะอาดด้านหลัง (Clean cotton supply well) เชื่อมต่อซิปเข้ากับม้วนแผ่นรอง และหมุนเลื่อนให้แผ่นใหม่เข้าสู่ตำแหน่งใต้เปลพยุง',
          cam: 'back'
        },
        {
          id: 5,
          shortTitle: 'ลดเปลกลับสู่ตำแหน่งพัก',
          title: 'ขั้นตอนที่ 5: ลดเปลกลับสู่ตำแหน่งพัก (Restore Resting Contact & Lock)',
          crankTag: 'กลไก: ยกฐานกลับขึ้นประกบเปล + ล็อกตำแหน่งพัก',
          lockText: 'สถานะ: ล็อกแผ่นรองและเปลแนบตำแหน่งพักแล้ว',
          desc: 'ล็อกแผ่นรองให้เข้าที่ จากนั้นหมุนมือหมุนปรับชุดแผ่นรองและเปลตาข่ายกลับสู่ตำแหน่งพักแนบสนิทกันอย่างนุ่มนวลให้สุนัขกลับมาพักบนแผ่นรองใหม่ได้อย่างปลอดภัย',
          cam: 'overview'
        }
      ];

      const CAM_PRESETS = {
        overview: { pos: [4.7, 3.3, 4.9], target: [0, 1.1, 0] },
        front:    { pos: [0.0, 2.2, 5.5], target: [0, 1.05, 0.4] },
        back:     { pos: [0.0, 2.4, -5.5], target: [0, 1.1, -0.3] },
        side:     { pos: [5.7, 2.1, 0.2], target: [0, 1.1, 0] }
      };

      // ==========================================
      // 2. STATE & UI REFERENCES
      // ==========================================
      let currentStepIndex = 0;
      let stepProgress = 0;
      let isPlaying = true;
      let autoAdvanceSteps = true;
      const STEP_DURATION = 4.2;

      const container = document.getElementById('canvas-container');
      const stepListEl = document.getElementById('step-list');
      const stepIndicatorNum = document.getElementById('step-indicator-num');
      const stepIndicatorTitle = document.getElementById('step-indicator-title');
      const lockStatusText = document.getElementById('lock-status-text');
      const detailBadge = document.getElementById('detail-badge');
      const detailCrankTag = document.getElementById('detail-crank-tag');
      const detailTitle = document.getElementById('detail-title');
      const detailDesc = document.getElementById('detail-desc');
      const progressBar = document.getElementById('progress-bar');
      const progressPct = document.getElementById('progress-pct');
      const playPauseBtn = document.getElementById('play-pause-btn');
      const playPauseIcon = document.getElementById('play-pause-icon');
      const playPauseLabel = document.getElementById('play-pause-label');
      const prevBtn = document.getElementById('prev-btn');
      const nextBtn = document.getElementById('next-btn');
      const resetViewBtn = document.getElementById('reset-view-btn');
      const camBtns = document.querySelectorAll('.cam-btn');

      // ==========================================
      // 3. THREE.JS SCENE SETUP
      // ==========================================
      let scene, camera, renderer;
      let desiredCamPos = new THREE.Vector3(...CAM_PRESETS.overview.pos);
      let desiredCamTarget = new THREE.Vector3(...CAM_PRESETS.overview.target);
      let currentCamTarget = new THREE.Vector3(...CAM_PRESETS.overview.target);

      let padCarrierGroup;
      let liftGroup;
      let slingFabric;
      let dogGroup;
      let liftCrankHandle;
      let feedCrankHandle;
      let activePadMesh;
      let zipperStrip;
      let usedPadMesh;
      let newPadMesh;
      let cleanRollMesh;
      let rearLidGroup;
      let lockIndicatorMesh;

      function createMeshTexture() {
        const c = document.createElement('canvas');
        c.width = 128;
        c.height = 128;
        const ctx = c.getContext('2d');
        ctx.fillStyle = '#0284c7';
        ctx.fillRect(0, 0, 128, 128);
        ctx.strokeStyle = '#38bdf8';
        ctx.lineWidth = 3;
        for (let i = -128; i < 256; i += 16) {
          ctx.beginPath();
          ctx.moveTo(i, 0);
          ctx.lineTo(i + 128, 128);
          ctx.stroke();
          ctx.beginPath();
          ctx.moveTo(i + 128, 0);
          ctx.lineTo(i, 128);
          ctx.stroke();
        }
        const tex = new THREE.CanvasTexture(c);
        tex.wrapS = THREE.RepeatWrapping;
        tex.wrapT = THREE.RepeatWrapping;
        tex.repeat.set(8, 12);
        return tex;
      }

      function buildStylizedDog() {
        const group = new THREE.Group();
        const furMat = new THREE.MeshStandardMaterial({ color: 0xd9a05b, roughness: 0.75 });
        const earMat = new THREE.MeshStandardMaterial({ color: 0xb87d3b, roughness: 0.8 });
        const noseMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.4 });

        // Torso (cylinder + two end spheres for r128 compatibility)
        const torsoCyl = new THREE.Mesh(new THREE.CylinderGeometry(0.32, 0.30, 0.90, 20), furMat);
        torsoCyl.rotation.x = Math.PI / 2;
        torsoCyl.position.set(0, 0.29, 0.05);
        torsoCyl.castShadow = true;
        group.add(torsoCyl);

        const chestSphere = new THREE.Mesh(new THREE.SphereGeometry(0.32, 20, 16), furMat);
        chestSphere.position.set(0, 0.29, 0.50);
        chestSphere.castShadow = true;
        group.add(chestSphere);

        const hipSphere = new THREE.Mesh(new THREE.SphereGeometry(0.30, 20, 16), furMat);
        hipSphere.position.set(0, 0.29, -0.40);
        hipSphere.castShadow = true;
        group.add(hipSphere);

        // Head & Snout
        const head = new THREE.Mesh(new THREE.SphereGeometry(0.24, 20, 16), furMat);
        head.position.set(0, 0.43, 0.72);
        head.castShadow = true;
        group.add(head);

        const snout = new THREE.Mesh(new THREE.CylinderGeometry(0.11, 0.14, 0.24, 16), furMat);
        snout.rotation.x = Math.PI / 2;
        snout.position.set(0, 0.38, 0.92);
        group.add(snout);

        const nose = new THREE.Mesh(new THREE.SphereGeometry(0.05, 12, 12), noseMat);
        nose.position.set(0, 0.41, 1.04);
        group.add(nose);

        // Floppy Retriever Ears
        [-0.21, 0.21].forEach((x) => {
          const ear = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.22, 0.16), earMat);
          ear.position.set(x, 0.40, 0.68);
          ear.rotation.z = x > 0 ? -0.25 : 0.25;
          group.add(ear);
        });

        // Resting Legs
        const legPositions = [
          [-0.22, 0.11,  0.58],
          [ 0.22, 0.11,  0.58],
          [-0.24, 0.11, -0.42],
          [ 0.24, 0.11, -0.42]
        ];
        legPositions.forEach(([x, y, z]) => {
          const leg = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.075, 0.52, 12), furMat);
          leg.rotation.x = Math.PI / 2;
          leg.rotation.z = x > 0 ? -0.2 : 0.2;
          leg.position.set(x, y, z + 0.14);
          leg.castShadow = true;
          group.add(leg);
        });

        // Tail
        const tail = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.07, 0.48, 12), furMat);
        tail.rotation.x = Math.PI / 2.4;
        tail.position.set(0.08, 0.22, -0.76);
        group.add(tail);

        return group;
      }

      function buildVareeMakiBed() {
        const bedW = 2.1;
        const bedL = 3.2;

        const frameMat = new THREE.MeshStandardMaterial({ color: 0x334155, metalness: 0.5, roughness: 0.35 });
        const postMat = new THREE.MeshStandardMaterial({ color: 0x475569, metalness: 0.45, roughness: 0.4 });
        const rubberBaseMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.75, metalness: 0.1 });
        const foamTopMat = new THREE.MeshStandardMaterial({ color: 0x0f766e, roughness: 0.65 });
        const cottonMat = new THREE.MeshStandardMaterial({ color: 0xfef3c7, roughness: 0.85 });
        const cleanNewMat = new THREE.MeshStandardMaterial({ color: 0x34d399, roughness: 0.75 });
        const usedPadMat = new THREE.MeshStandardMaterial({ color: 0xd97706, roughness: 0.85 });
        const zipperMat = new THREE.MeshStandardMaterial({ color: 0x0ea5e9, metalness: 0.7, roughness: 0.3 });

        // 1. Four outer corner posts & lower crossbars (standing on floor, keeping bed elevated)
        const postPositions = [
          [-bedW / 2 + 0.08, 0.92, -bedL / 2 + 0.08],
          [ bedW / 2 - 0.08, 0.92, -bedL / 2 + 0.08],
          [-bedW / 2 + 0.08, 0.92,  bedL / 2 - 0.08],
          [ bedW / 2 - 0.08, 0.92,  bedL / 2 - 0.08]
        ];
        postPositions.forEach(([x, y, z]) => {
          const post = new THREE.Mesh(new THREE.CylinderGeometry(0.085, 0.095, 1.84, 16), postMat);
          post.position.set(x, y, z);
          post.castShadow = true;
          scene.add(post);

          const foot = new THREE.Mesh(new THREE.CylinderGeometry(0.11, 0.12, 0.06, 16), rubberBaseMat);
          foot.position.set(x, 0.03, z);
          scene.add(foot);
        });

        // Outer side rails at top and bottom (leaving clear open space above floor)
        [-bedW / 2 + 0.08, bedW / 2 - 0.08].forEach((x) => {
          const topRail = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.06, bedL - 0.16), frameMat);
          topRail.position.set(x, 1.15, 0);
          scene.add(topRail);

          const bottomBrace = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.05, bedL - 0.16), frameMat);
          bottomBrace.position.set(x, 0.22, 0);
          scene.add(bottomBrace);
        });

        // 2. Lowerable Underpad & Soft Base Carrier Group
        padCarrierGroup = new THREE.Group();
        padCarrierGroup.position.set(0, 0.74, 0);

        const baseTray = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.16, 0.22, bedL - 0.12), rubberBaseMat);
        baseTray.position.set(0, 0.11, 0);
        baseTray.castShadow = true;
        baseTray.receiveShadow = true;
        padCarrierGroup.add(baseTray);

        const softTop = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.32, 0.12, bedL - 0.58), foamTopMat);
        softTop.position.set(0, 0.27, 0.05);
        softTop.receiveShadow = true;
        padCarrierGroup.add(softTop);

        const frontSlotHousing = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.24, 0.24, 0.22), frameMat);
        frontSlotHousing.position.set(0, 0.36, bedL / 2 - 0.12);
        frontSlotHousing.castShadow = true;
        padCarrierGroup.add(frontSlotHousing);

        const backWellHousing = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.22, 0.42, 0.44), frameMat);
        backWellHousing.position.set(0, 0.45, -bedL / 2 + 0.18);
        backWellHousing.castShadow = true;
        padCarrierGroup.add(backWellHousing);

        rearLidGroup = new THREE.Group();
        rearLidGroup.position.set(0, 0.67, -bedL / 2);
        const lidMesh = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.28, 0.05, 0.40), postMat);
        lidMesh.position.set(0, 0.025, 0.18);
        lidMesh.castShadow = true;
        rearLidGroup.add(lidMesh);
        padCarrierGroup.add(rearLidGroup);

        cleanRollMesh = new THREE.Mesh(
          new THREE.CylinderGeometry(0.13, 0.13, bedW - 0.40, 20),
          cottonMat
        );
        cleanRollMesh.rotation.z = Math.PI / 2;
        cleanRollMesh.position.set(0, 0.47, -bedL / 2 + 0.18);
        padCarrierGroup.add(cleanRollMesh);

        // Hand Cranks attached to carrier side (+X)
        const liftGear = new THREE.Mesh(new THREE.CylinderGeometry(0.085, 0.085, 0.06, 16), frameMat);
        liftGear.rotation.z = Math.PI / 2;
        liftGear.position.set(bedW / 2 - 0.04, 0.22, 0.65);
        padCarrierGroup.add(liftGear);

        liftCrankHandle = new THREE.Group();
        liftCrankHandle.position.set(bedW / 2, 0.22, 0.65);
        const arm1 = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.22, 0.04), new THREE.MeshStandardMaterial({ color: 0x38bdf8, metalness: 0.4, roughness: 0.3 }));
        arm1.position.y = 0.09;
        const grip1 = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.03, 0.14, 12), rubberBaseMat);
        grip1.rotation.z = Math.PI / 2;
        grip1.position.set(0.08, 0.18, 0);
        liftCrankHandle.add(arm1, grip1);
        padCarrierGroup.add(liftCrankHandle);

        const feedGear = new THREE.Mesh(new THREE.CylinderGeometry(0.085, 0.085, 0.06, 16), frameMat);
        feedGear.rotation.z = Math.PI / 2;
        feedGear.position.set(bedW / 2 - 0.04, 0.22, -0.65);
        padCarrierGroup.add(feedGear);

        feedCrankHandle = new THREE.Group();
        feedCrankHandle.position.set(bedW / 2, 0.22, -0.65);
        const arm2 = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.22, 0.04), new THREE.MeshStandardMaterial({ color: 0xfbbf24, metalness: 0.4, roughness: 0.3 }));
        arm2.position.y = 0.09;
        const grip2 = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.03, 0.14, 12), rubberBaseMat);
        grip2.rotation.z = Math.PI / 2;
        grip2.position.set(0.08, 0.18, 0);
        feedCrankHandle.add(arm2, grip2);
        padCarrierGroup.add(feedCrankHandle);

        activePadMesh = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.44, 0.035, 1.82), cottonMat);
        activePadMesh.position.set(0, 0.35, 0.05);
        activePadMesh.receiveShadow = true;
        padCarrierGroup.add(activePadMesh);

        zipperStrip = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.46, 0.042, 0.04), zipperMat);
        zipperStrip.position.set(0, 0.352, -0.75);
        padCarrierGroup.add(zipperStrip);

        usedPadMesh = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.50, 0.03, 1.12), usedPadMat);
        usedPadMesh.position.set(0, 0.36, bedL / 2 + 0.12);
        usedPadMesh.castShadow = true;
        usedPadMesh.visible = false;
        padCarrierGroup.add(usedPadMesh);

        newPadMesh = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.46, 0.04, 1.52), cleanNewMat);
        newPadMesh.position.set(0, 0.85, -bedL / 2 - 0.55);
        newPadMesh.castShadow = true;
        newPadMesh.visible = false;
        padCarrierGroup.add(newPadMesh);

        lockIndicatorMesh = new THREE.Mesh(
          new THREE.BoxGeometry(0.12, 0.09, 0.14),
          new THREE.MeshStandardMaterial({ color: 0x10b981, emissive: 0x059669, emissiveIntensity: 0.4 })
        );
        lockIndicatorMesh.position.set(bedW / 2 - 0.14, 0.34, 0.85);
        padCarrierGroup.add(lockIndicatorMesh);

        scene.add(padCarrierGroup);

        // 3. Top Breathable Mesh Sling + Stylized Dog (stays at y = 1.15)
        liftGroup = new THREE.Group();
        liftGroup.position.set(0, 1.15, 0);

        [-bedW / 2 + 0.17, bedW / 2 - 0.17].forEach((x) => {
          const slingBar = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.055, bedL - 0.35), frameMat);
          slingBar.position.set(x, 0, 0);
          slingBar.castShadow = true;
          liftGroup.add(slingBar);
        });

        const meshTex = createMeshTexture();
        const slingMat = new THREE.MeshStandardMaterial({
          map: meshTex,
          transparent: true,
          opacity: 0.92,
          roughness: 0.5,
          side: THREE.DoubleSide
        });
        slingFabric = new THREE.Mesh(new THREE.BoxGeometry(bedW - 0.36, 0.025, bedL - 0.45), slingMat);
        slingFabric.position.set(0, 0.01, 0);
        slingFabric.receiveShadow = true;
        liftGroup.add(slingFabric);

        dogGroup = buildStylizedDog();
        dogGroup.position.set(0, 0.02, 0.05);
        liftGroup.add(dogGroup);

        scene.add(liftGroup);
      }

      function initThree() {
        scene = new THREE.Scene();
        scene.background = new THREE.Color(0x0b1220);
        scene.fog = new THREE.Fog(0x0b1220, 14, 28);

        const width = container.clientWidth || 800;
        const height = container.clientHeight || 460;

        camera = new THREE.PerspectiveCamera(42, width / height, 0.1, 100);
        camera.position.copy(desiredCamPos);

        renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
        renderer.setSize(width, height);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        container.innerHTML = '';
        container.appendChild(renderer.domElement);

        // Lighting
        const hemiLight = new THREE.HemisphereLight(0xe0f2fe, 0x1e293b, 0.85);
        scene.add(hemiLight);

        const dirLight = new THREE.DirectionalLight(0xffffff, 1.15);
        dirLight.position.set(5, 8, 6);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        dirLight.shadow.camera.near = 1;
        dirLight.shadow.camera.far = 25;
        dirLight.shadow.camera.left = -4;
        dirLight.shadow.camera.right = 4;
        dirLight.shadow.camera.top = 4;
        dirLight.shadow.camera.bottom = -4;
        scene.add(dirLight);

        const fillLight = new THREE.DirectionalLight(0x38bdf8, 0.45);
        fillLight.position.set(-5, 4, -5);
        scene.add(fillLight);

        // Studio Ground Plane & Subtle Grid
        const ground = new THREE.Mesh(
          new THREE.PlaneGeometry(30, 30),
          new THREE.MeshStandardMaterial({ color: 0x0f172a, roughness: 0.9, metalness: 0.05 })
        );
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        const grid = new THREE.GridHelper(16, 24, 0x1e293b, 0x162033);
        grid.position.y = 0.002;
        scene.add(grid);

        buildVareeMakiBed();
        setupPointerControls();
        window.addEventListener('resize', handleResize);
      }

      // ==========================================
      // 4. ANIMATION & STEP POSE INTERPOLATION
      // ==========================================
      function easeInOut(t) {
        return t < 0.5 ? 2 * t * t : 1 - Math.pow(-2 * t + 2, 2) / 2;
      }

      function applyStepPose(stepIdx, t) {
        const e = easeInOut(t);

        let carrierY = 0.74;
        let slingY = 1.15;
        let dogBob = 0;
        let liftCrankAngle = 0;
        let feedCrankAngle = 0;
        let lidAngle = 0;
        let locked = false;

        activePadMesh.visible = true;
        activePadMesh.position.set(0, 0.35, 0.05);
        zipperStrip.visible = true;
        zipperStrip.position.set(0, 0.352, -0.75);
        usedPadMesh.visible = false;
        newPadMesh.visible = false;

        if (stepIdx === 0) {
          // Step 1: Resting — sling flush against underpad & soft base elevated off floor
          carrierY = 0.74;
          dogBob = Math.sin(t * Math.PI * 2) * 0.006;
        } else if (stepIdx === 1) {
          // Step 2: Clear solid waste & lower the underpad/soft base assembly downward + lock frame
          carrierY = 0.74 - e * 0.42;
          liftCrankAngle = -e * Math.PI * 4;
          locked = e > 0.75;
        } else if (stepIdx === 2) {
          // Step 3: Carrier stays lowered; feed crank turns; used pad exits front & detaches
          carrierY = 0.32;
          locked = true;
          liftCrankAngle = -Math.PI * 4;
          feedCrankAngle = e * Math.PI * 5;

          activePadMesh.position.z = 0.05 + e * 0.25;
          zipperStrip.position.z = -0.75 + e * 0.25;
          usedPadMesh.visible = true;
          usedPadMesh.position.set(0, 0.36 - e * 0.14, 1.72 + e * 0.85);
          usedPadMesh.rotation.x = e * 0.25;
        } else if (stepIdx === 3) {
          // Step 4: Bring new clean pad into rear supply well, zip connect & feed into place
          carrierY = 0.32;
          locked = true;
          liftCrankAngle = -Math.PI * 4;
          feedCrankAngle = Math.PI * 5 + e * Math.PI * 4;

          lidAngle = e < 0.5 ? -(e * 2) * 1.15 : -((1 - e) * 2) * 1.15;

          newPadMesh.visible = true;
          const startZ = -2.35;
          const endZ = 0.05;
          const startY = 1.05;
          const endY = 0.355;
          newPadMesh.position.set(
            0,
            startY + (endY - startY) * Math.min(1, e * 1.3),
            startZ + (endZ - startZ) * e
          );
          activePadMesh.position.z = 0.30 - e * 0.25;
          zipperStrip.position.z = -0.50 - e * 0.25;
          cleanRollMesh.rotation.x = -e * Math.PI * 3;
        } else if (stepIdx === 4) {
          // Step 5: Raise pad carrier back up flush to mesh sling & lock resting position
          carrierY = 0.32 + e * 0.42;
          liftCrankAngle = -Math.PI * 4 + e * Math.PI * 4;
          feedCrankAngle = Math.PI * 9;
          locked = true;
          newPadMesh.visible = true;
          newPadMesh.position.set(0, 0.355, 0.05);
          activePadMesh.visible = false;
          dogBob = -Math.sin(e * Math.PI) * 0.008;
        }

        padCarrierGroup.position.y = carrierY;
        liftGroup.position.y = slingY;
        dogGroup.position.y = 0.02 + dogBob;
        liftCrankHandle.rotation.x = liftCrankAngle;
        feedCrankHandle.rotation.x = feedCrankAngle;
        rearLidGroup.rotation.x = lidAngle;

        if (locked) {
          lockIndicatorMesh.material.color.setHex(0x10b981);
          lockIndicatorMesh.material.emissive.setHex(0x059669);
        } else {
          lockIndicatorMesh.material.color.setHex(0xf59e0b);
          lockIndicatorMesh.material.emissive.setHex(0xb45309);
        }
      }

      // ==========================================
      // 5. UI SYNCHRONIZATION & CONTROLS
      // ==========================================
      function renderStepButtons() {
        stepListEl.innerHTML = '';
        STEPS.forEach((step, idx) => {
          const isActive = idx === currentStepIndex;
          const btn = document.createElement('button');
          btn.type = 'button';
          btn.className = isActive
            ? 'w-full text-left p-3 rounded-xl bg-teal-500/15 border border-teal-400/60 shadow-md transition flex items-start gap-3'
            : 'w-full text-left p-3 rounded-xl bg-slate-800/70 hover:bg-slate-800 border border-slate-700/70 transition flex items-start gap-3';

          btn.innerHTML = `
            <span class="w-6 h-6 rounded-lg flex items-center justify-center text-xs font-bold shrink-0 mt-0.5 ${
              isActive ? 'bg-teal-400 text-slate-950' : 'bg-slate-700 text-slate-200'
            }">${step.id}</span>
            <div class="min-w-0 flex-1">
              <div class="text-xs sm:text-sm font-semibold ${isActive ? 'text-white' : 'text-slate-200'} truncate">
                ${step.shortTitle}
              </div>
              <div class="text-[11px] text-slate-400 truncate mt-0.5">
                ${step.crankTag}
              </div>
            </div>
          `;

          btn.addEventListener('click', () => {
            selectStep(idx, true);
          });
          stepListEl.appendChild(btn);
        });
      }

      function syncStepUI() {
        const step = STEPS[currentStepIndex];
        stepIndicatorNum.textContent = `ขั้นตอนที่ ${step.id} / ${STEPS.length}`;
        stepIndicatorTitle.textContent = step.title;
        lockStatusText.textContent = step.lockText;

        detailBadge.textContent = `ขั้นตอนที่ ${step.id}`;
        detailCrankTag.textContent = step.crankTag;
        detailTitle.textContent = step.title;
        detailDesc.textContent = step.desc;

        const overallPct = Math.round(((currentStepIndex + stepProgress) / STEPS.length) * 100);
        progressBar.style.width = `${overallPct}%`;
        progressPct.textContent = `${overallPct}%`;

        prevBtn.disabled = currentStepIndex === 0 && stepProgress <= 0.05;
        nextBtn.disabled = currentStepIndex === STEPS.length - 1 && stepProgress >= 0.99;

        renderStepButtons();
      }

      function updatePlayPauseUI() {
        if (isPlaying) {
          playPauseIcon.textContent = '⏸';
          playPauseLabel.textContent = 'หยุดชั่วคราว';
        } else {
          playPauseIcon.textContent = '▶';
          playPauseLabel.textContent = 'เล่นต่อ';
        }
      }

      function setCameraPreset(presetKey) {
        const preset = CAM_PRESETS[presetKey] || CAM_PRESETS.overview;
        desiredCamPos.set(...preset.pos);
        desiredCamTarget.set(...preset.target);

        camBtns.forEach((b) => {
          const active = b.dataset.cam === presetKey;
          b.className = active
            ? 'cam-btn px-3 py-1.5 rounded-lg text-xs font-medium bg-teal-600 text-white border border-teal-400/50 transition'
            : 'cam-btn px-3 py-1.5 rounded-lg text-xs font-medium bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 transition';
        });
      }

      function selectStep(idx, shouldPlay = true) {
        currentStepIndex = Math.max(0, Math.min(STEPS.length - 1, idx));
        stepProgress = 0;
        isPlaying = shouldPlay;
        updatePlayPauseUI();
        setCameraPreset(STEPS[currentStepIndex].cam);
        syncStepUI();
      }

      playPauseBtn.addEventListener('click', () => {
        if (!isPlaying && currentStepIndex === STEPS.length - 1 && stepProgress >= 0.99) {
          currentStepIndex = 0;
          stepProgress = 0;
          setCameraPreset(STEPS[0].cam);
        }
        isPlaying = !isPlaying;
        updatePlayPauseUI();
      });

      prevBtn.addEventListener('click', () => {
        if (stepProgress > 0.25) {
          stepProgress = 0;
          syncStepUI();
        } else if (currentStepIndex > 0) {
          selectStep(currentStepIndex - 1, true);
        }
      });

      nextBtn.addEventListener('click', () => {
        if (currentStepIndex < STEPS.length - 1) {
          selectStep(currentStepIndex + 1, true);
        } else {
          stepProgress = 1;
          isPlaying = false;
          updatePlayPauseUI();
          syncStepUI();
        }
      });

      resetViewBtn.addEventListener('click', () => {
        setCameraPreset(STEPS[currentStepIndex].cam);
      });

      camBtns.forEach((btn) => {
        btn.addEventListener('click', () => {
          setCameraPreset(btn.dataset.cam);
        });
      });

      // ==========================================
      // 6. ORBIT / DRAG / ZOOM CONTROLS (MOUSE & TOUCH)
      // ==========================================
      function setupPointerControls() {
        let isDragging = false;
        let prevX = 0;
        let prevY = 0;
        let lastPinchDist = null;

        const dom = renderer.domElement;

        dom.addEventListener('mousedown', (e) => {
          isDragging = true;
          prevX = e.clientX;
          prevY = e.clientY;
        });

        window.addEventListener('mouseup', () => {
          isDragging = false;
        });

        window.addEventListener('mousemove', (e) => {
          if (!isDragging) return;
          const dx = e.clientX - prevX;
          const dy = e.clientY - prevY;
          prevX = e.clientX;
          prevY = e.clientY;
          orbitCamera(dx * 0.008, dy * 0.008);
        });

        dom.addEventListener('wheel', (e) => {
          e.preventDefault();
          zoomCamera(e.deltaY * 0.004);
        }, { passive: false });

        dom.addEventListener('touchstart', (e) => {
          if (e.touches.length === 1) {
            isDragging = true;
            prevX = e.touches[0].clientX;
            prevY = e.touches[0].clientY;
            lastPinchDist = null;
          } else if (e.touches.length === 2) {
            isDragging = false;
            const dx = e.touches[0].clientX - e.touches[1].clientX;
            const dy = e.touches[0].clientY - e.touches[1].clientY;
            lastPinchDist = Math.hypot(dx, dy);
          }
        }, { passive: true });

        dom.addEventListener('touchmove', (e) => {
          if (e.touches.length === 1 && isDragging) {
            const dx = e.touches[0].clientX - prevX;
            const dy = e.touches[0].clientY - prevY;
            prevX = e.touches[0].clientX;
            prevY = e.touches[0].clientY;
            orbitCamera(dx * 0.009, dy * 0.009);
          } else if (e.touches.length === 2 && lastPinchDist !== null) {
            const dx = e.touches[0].clientX - e.touches[1].clientX;
            const dy = e.touches[0].clientY - e.touches[1].clientY;
            const dist = Math.hypot(dx, dy);
            const delta = (lastPinchDist - dist) * 0.015;
            zoomCamera(delta);
            lastPinchDist = dist;
          }
        }, { passive: true });

        dom.addEventListener('touchend', () => {
          isDragging = false;
          lastPinchDist = null;
        });
      }

      function orbitCamera(yawDelta, pitchDelta) {
        const offset = camera.position.clone().sub(desiredCamTarget);
        const spherical = new THREE.Spherical().setFromVector3(offset);
        spherical.theta -= yawDelta;
        spherical.phi = Math.max(0.22, Math.min(Math.PI / 2 - 0.05, spherical.phi - pitchDelta));
        offset.setFromSpherical(spherical);
        desiredCamPos.copy(desiredCamTarget).add(offset);
      }

      function zoomCamera(delta) {
        const offset = camera.position.clone().sub(desiredCamTarget);
        const len = Math.max(2.6, Math.min(11.5, offset.length() + delta));
        offset.setLength(len);
        desiredCamPos.copy(desiredCamTarget).add(offset);
      }

      function handleResize() {
        if (!renderer || !camera) return;
        const w = container.clientWidth || 800;
        const h = container.clientHeight || 460;
        camera.aspect = w / h;
        camera.updateProjectionMatrix();
        renderer.setSize(w, h);
      }

      // ==========================================
      // 7. MAIN ANIMATION LOOP
      // ==========================================
      let lastTime = performance.now();

      function animate(now) {
        requestAnimationFrame(animate);
        const dt = Math.min(0.1, (now - lastTime) / 1000);
        lastTime = now;

        if (isPlaying) {
          stepProgress += dt / STEP_DURATION;
          if (stepProgress >= 1) {
            if (autoAdvanceSteps && currentStepIndex < STEPS.length - 1) {
              currentStepIndex += 1;
              stepProgress = 0;
              setCameraPreset(STEPS[currentStepIndex].cam);
            } else {
              stepProgress = 1;
              isPlaying = false;
              updatePlayPauseUI();
            }
          }
          syncStepUI();
        }

        applyStepPose(currentStepIndex, stepProgress);

        camera.position.lerp(desiredCamPos, 1 - Math.exp(-6 * dt));
        currentCamTarget.lerp(desiredCamTarget, 1 - Math.exp(-6 * dt));
        camera.lookAt(currentCamTarget);

        renderer.render(scene, camera);
      }

      window.addEventListener('load', () => {
        initThree();
        syncStepUI();
        animate(performance.now());
      });
    })();
  </script>
</body>
</html>
