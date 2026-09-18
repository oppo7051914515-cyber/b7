<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Check-In Online - ระบบเช็คชื่อนักเรียนข้ามอุปกรณ์ผ่าน QR Code และ OTP</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts Prompt -->
    <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- QRCode.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        prompt: ['Prompt', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#f0f9ff',
                            100: '#e0f2fe',
                            500: '#0284c7',
                            600: '#0284c7',
                            700: '#0369a1',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body { font-family: 'Prompt', sans-serif; }
        .glass-card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(226, 232, 240, 0.8);
        }
        .animate-pulse-slow {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col">

    <!-- Navigation Bar -->
    <header class="bg-slate-900 text-white shadow-lg sticky top-0 z-40">
        <div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-3 cursor-pointer" onclick="switchTab('dashboard')">
                <div class="bg-blue-600 p-2.5 rounded-xl text-white shadow-md flex items-center justify-center">
                    <i class="fa-solid fa-qrcode text-xl"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg leading-tight">Smart Check-In</h1>
                    <p class="text-xs text-slate-400">ระบบเช็คชื่อนักเรียนออนไลน์ข้ามอุปกรณ์ Realtime</p>
                </div>
            </div>
            
            <div id="teacher-nav" class="hidden md:flex space-x-1 text-sm font-medium">
                <button onclick="switchTab('dashboard')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-blue-400" id="nav-dashboard">
                    <i class="fa-solid fa-chalkboard-user mr-1.5"></i>เปิดคาบเรียน
                </button>
                <button onclick="switchTab('classes')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-classes">
                    <i class="fa-solid fa-school mr-1.5"></i>จัดการห้องเรียน
                </button>
                <button onclick="switchTab('students')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-students">
                    <i class="fa-solid fa-users mr-1.5"></i>จัดการนักเรียน
                </button>
                <button onclick="switchTab('reports')" class="nav-btn px-4 py-2 rounded-lg hover:bg-slate-800 transition text-slate-300" id="nav-reports">
                    <i class="fa-solid fa-chart-line mr-1.5"></i>รายงานสถิติ
                </button>
            </div>

            <div class="flex items-center space-x-2">
                <span id="sync-status" class="inline-flex items-center text-xs px-3 py-1.5 rounded-full bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 font-medium">
                    <span class="w-2 h-2 rounded-full bg-emerald-400 mr-1.5 animate-pulse">
                    </span> Cloud Online DB Ready
                </span>
            </div>
        </div>
        
        <!-- Mobile Navigation Menu -->
        <div id="teacher-nav-mobile" class="md:hidden flex justify-around border-t border-slate-800 py-2.5 text-xs bg-slate-900/95 backdrop-blur">
            <button onclick="switchTab('dashboard')" class="text-blue-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-chalkboard-user text-base mb-1"></i>เช็คชื่อ
            </button>
            <button onclick="switchTab('classes')" class="text-slate-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-school text-base mb-1"></i>ห้องเรียน
            </button>
            <button onclick="switchTab('students')" class="text-slate-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-users text-base mb-1"></i>นักเรียน
            </button>
            <button onclick="switchTab('reports')" class="text-slate-400 flex flex-col items-center font-medium">
                <i class="fa-solid fa-chart-line text-base mb-1"></i>รายงาน
            </button>
        </div>
    </header>

    <!-- Main Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto p-4 md:p-6">

        <!-- STUDENT VIEW (Visible when scanned via URL query param e.g. ?session=...) -->
        <div id="student-view" class="hidden max-w-md mx-auto">
            <div class="glass-card rounded-2xl shadow-xl p-6 text-center border-t-4 border-blue-600">
                <div class="w-16 h-16 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center mx-auto mb-4 text-2xl shadow-inner">
                    <i class="fa-solid fa-user-check"></i>
                </div>
                <h2 class="text-2xl font-bold text-slate-800" id="student-class-title">กำลังโหลดข้อมูลคาบเรียน...</h2>
                <p class="text-sm text-slate-500 mt-1 mb-6" id="student-session-info">กำลังเชื่อมต่อฐานข้อมูลคลาวด์...</p>

                <!-- Student Selection Box -->
                <div id="student-step-select" class="space-y-4">
                    <div class="text-left">
                        <label class="block text-xs font-semibold text-slate-600 uppercase mb-2">เลือกชื่อนักเรียนของคุณ</label>
                        <select id="student-dropdown" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-3 text-slate-800 focus:ring-2 focus:ring-blue-500 focus:outline-none shadow-sm">
                            <option value="">-- กรุณาเลือกรายชื่อ --</option>
                        </select>
                    </div>

                    <button onclick="requestStudentOTP()" class="w-full bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-700 hover:to-indigo-700 text-white font-semibold py-3.5 px-4 rounded-xl shadow-lg shadow-blue-500/30 transition duration-200 active:scale-95 flex items-center justify-center space-x-2">
                        <i class="fa-solid fa-key"></i>
                        <span>ขอรับรหัส OTP 6 หลัก</span>
                    </button>
                </div>

                <!-- Student OTP Display Box -->
                <div id="student-step-otp" class="hidden mt-6 bg-slate-900 text-white rounded-2xl p-6 shadow-inner relative overflow-hidden">
                    <div class="absolute -right-6 -bottom-6 text-slate-800 opacity-20 text-8xl font-black">OTP</div>
                    <span class="inline-block bg-amber-500/20 text-amber-300 text-xs px-3 py-1 rounded-full font-medium mb-3 border border-amber-500/30">
                        <i class="fa-solid fa-clock mr-1 animate-spin"></i>รอคุณครูทำการยืนยัน
                    </span>
                    <p class="text-xs text-slate-400">รหัส OTP ของคุณคือ</p>
                    <div class="text-4xl font-extrabold tracking-widest text-amber-400 my-3 font-mono" id="display-otp-code">
                        ------
                    </div>
                    <p class="text-xs text-slate-300 bg-slate-800/80 p-2.5 rounded-lg border border-slate-700 leading-relaxed">
                        💡 โปรดแจ้งรหัส 6 หลักนี้แก่ครูผู้สอนเพื่อยืนยันการเข้าเรียน
                    </p>
                    <div class="mt-4 pt-4 border-t border-slate-800 flex justify-between items-center text-xs text-slate-400">
                        <span>สถานะปัจจุบัน:</span>
                        <span id="student-status-badge" class="font-semibold text-amber-400">🟡 รอครูอนุมัติ...</span>
                    </div>
                </div>

                <!-- Student Success State -->
                <div id="student-step-success" class="hidden mt-6 bg-emerald-50 text-emerald-800 rounded-2xl p-6 border border-emerald-200 shadow-sm text-center">
                    <div class="w-14 h-14 bg-emerald-500 text-white rounded-full flex items-center justify-center mx-auto mb-3 text-2xl shadow-lg">
                        <i class="fa-solid fa-check"></i>
                    </div>
                    <h3 class="font-bold text-xl">เช็คชื่อสำเร็จแล้ว!</h3>
                    <p class="text-xs text-emerald-600 mt-1">คุณครูยืนยันการเข้าเรียนเรียบร้อยแล้ว ขอบคุณครับ</p>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 1 - DASHBOARD / LIVE SESSION -->
        <div id="tab-dashboard" class="tab-content space-y-6">
            <!-- Top Controls Card -->
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 items-end">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1.5 uppercase">1. เลือกห้องเรียน</label>
                        <select id="teacher-class-select" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-slate-800 focus:ring-2 focus:ring-blue-500 focus:outline-none">
                            <option value="">-- เลือกห้องเรียน --</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1.5 uppercase">2. ชื่อวิชา / คาบเรียน</label>
                        <input type="text" id="teacher-subject-input" placeholder="เช่น วิทยาการคำนวณ คาบ 1" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-slate-800 focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    </div>
                    <div>
                        <button id="btn-toggle-session" onclick="toggleClassSession()" class="w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold p-2.5 rounded-xl shadow-md transition flex items-center justify-center space-x-2">
                            <i class="fa-solid fa-play"></i>
                            <span>🟢 เริ่มเปิดคาบเรียนออนไลน์ (Cloud Sync)</span>
                        </button>
                    </div>
                </div>
            </div>

            <!-- Active Session Dashboard Grid -->
            <div id="active-session-container" class="hidden grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- QR Code & Quick OTP Panel -->
                <div class="lg:col-span-1 space-y-6">
                    <!-- QR Box -->
                    <div class="glass-card rounded-2xl p-6 shadow-sm text-center border-t-4 border-blue-600">
                        <span class="bg-emerald-100 text-emerald-800 text-xs px-3 py-1 rounded-full font-semibold inline-block mb-3 border border-emerald-200">
                            🔴 LIVE ONLINE CLOUD
                        </span>
                        <h3 class="font-bold text-slate-800 text-xl" id="live-class-name">ห้องเรียน</h3>
                        <p class="text-xs text-slate-500 mb-4" id="live-subject-name">วิชา</p>
                        
                        <!-- QR Code Container -->
                        <div class="bg-white p-4 rounded-2xl shadow-inner inline-block border border-slate-200">
                            <div id="qrcode" class="flex justify-center"></div>
                        </div>
                        <p class="text-xs text-slate-500 mt-3 font-medium">ให้นักเรียนสแกน QR Code เพื่อเลือกชื่อและขอ OTP</p>
                        
                        <div class="mt-4 pt-4 border-t border-slate-100">
                            <input type="text" id="session-link-input" readonly class="w-full text-xs bg-slate-100 border border-slate-200 rounded-lg p-2.5 text-slate-600 font-mono text-center mb-2 focus:outline-none">
                            <button onclick="copySessionLink()" class="w-full text-xs bg-blue-50 hover:bg-blue-100 text-blue-600 font-semibold py-2 px-3 rounded-lg border border-blue-200 transition flex items-center justify-center space-x-1.5">
                                <i class="fa-solid fa-copy"></i>
                                <span>คัดลอกลิงก์ให้นักเรียน</span>
                            </button>
                        </div>
                    </div>

                    <!-- Quick OTP Confirm Form -->
                    <div class="glass-card rounded-2xl p-6 shadow-sm bg-slate-900 text-white">
                        <h4 class="font-bold text-base mb-2 flex items-center">
                            <i class="fa-solid fa-shield-halved text-amber-400 mr-2"></i> กรอก OTP ยืนยันการเข้าเรียน
                        </h4>
                        <p class="text-xs text-slate-400 mb-4">เมื่อนักเรียนแจ้ง OTP 6 หลัก คุณครูสามารถพิมพ์ยืนยันที่นี่ได้ทันที</p>
                        <div class="flex space-x-2">
                            <input type="text" id="teacher-otp-input" placeholder="000000" maxlength="6" class="w-full text-center tracking-widest font-mono text-xl bg-slate-800 border border-slate-700 rounded-xl p-2.5 text-amber-400 focus:outline-none focus:ring-2 focus:ring-amber-400">
                            <button onclick="verifyTeacherOTP()" class="bg-amber-500 hover:bg-amber-600 active:scale-95 text-slate-950 font-bold px-4 rounded-xl transition shadow-lg flex items-center justify-center">
                                ยืนยัน
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Realtime Student Status Table -->
                <div class="lg:col-span-2">
                    <div class="glass-card rounded-2xl p-6 shadow-sm min-h-full flex flex-col justify-between">
                        <div>
                            <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-2">
                                <div>
                                    <h3 class="font-bold text-lg text-slate-800 flex items-center">
                                        สถานะการเช็คชื่อ Real-time
                                        <span class="ml-2 w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                                    </h3>
                                    <p class="text-xs text-slate-500">ซิงค์ข้อมูลกับ Cloud อัตโนมัติทุกๆ 2 วินาที</p>
                                </div>
                                <div class="flex space-x-2 text-xs font-semibold">
                                    <span class="px-2.5 py-1 rounded-md bg-red-100 text-red-700">🔴 ยังไม่เช็ค: <span id="cnt-absent">0</span></span>
                                    <span class="px-2.5 py-1 rounded-md bg-amber-100 text-amber-800">🟡 รออนุมัติ: <span id="cnt-pending">0</span></span>
                                    <span class="px-2.5 py-1 rounded-md bg-emerald-100 text-emerald-800">🟢 เข้าเรียน: <span id="cnt-present">0</span></span>
                                </div>
                            </div>

                            <!-- Table -->
                            <div class="overflow-x-auto rounded-xl border border-slate-200">
                                <table class="w-full text-left border-collapse text-sm">
                                    <thead class="bg-slate-100 text-slate-600 font-semibold text-xs uppercase">
                                        <tr>
                                            <th class="p-3 border-b">เลขที่</th>
                                            <th class="p-3 border-b">รหัสนักเรียน</th>
                                            <th class="p-3 border-b">ชื่อ - นามสกุล</th>
                                            <th class="p-3 border-b text-center">สถานะ</th>
                                            <th class="p-3 border-b text-center">OTP</th>
                                            <th class="p-3 border-b text-center">จัดการ</th>
                                        </tr>
                                    </thead>
                                    <tbody id="live-students-tbody" class="divide-y divide-slate-100 bg-white">
                                        <tr>
                                            <td colspan="6" class="text-center py-8 text-slate-400">กำลังดึงข้อมูลนักเรียนจาก Cloud Database...</td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>

                        <div class="mt-4 pt-4 border-t border-slate-100 flex justify-between items-center text-xs text-slate-400">
                            <span>Cloud DB Rest Sync Status: Active</span>
                            <span class="font-mono text-emerald-600"><i class="fa-solid fa-arrows-rotate animate-spin mr-1"></i> Polling Every 2s</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 2 - CLASS MANAGEMENT -->
        <div id="tab-classes" class="tab-content hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <h3 class="font-bold text-lg mb-4 text-slate-800">เพิ่มห้องเรียนใหม่</h3>
                <form id="form-add-class" onsubmit="addClass(event)" class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                    <input type="text" id="input-class-name" placeholder="ชื่อห้องเรียน (เช่น ม.4/1)" required class="bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    <input type="text" id="input-class-desc" placeholder="คำอธิบายเพิ่มเติม (ตัวอย่าง: ปีการศึกษา 2569)" class="bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none">
                    <button type="submit" class="bg-blue-600 hover:bg-blue-700 active:scale-95 text-white font-semibold p-2.5 rounded-xl shadow transition">
                        <i class="fa-solid fa-plus mr-1"></i> เพิ่มห้องเรียน
                    </button>
                </form>
            </div>

            <div class="glass-card rounded-2xl p-6 shadow-sm">
                <h3 class="font-bold text-lg mb-4 text-slate-800">รายชื่อห้องเรียนทั้งหมด</h3>
                <div class="overflow-x-auto rounded-xl border border-slate-200">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-slate-100 text-slate-600 font-semibold text-xs uppercase">
                            <tr>
                                <th class="p-3 border-b">ลำดับ</th>
                                <th class="p-3 border-b">ชื่อห้องเรียน</th>
                                <th class="p-3 border-b">คำอธิบาย</th>
                                <th class="p-3 border-b text-center">จำนวนนักเรียน</th>
                                <th class="p-3 border-b text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="classes-table-tbody" class="divide-y divide-slate-100 bg-white">
                            <!-- Class Items Dynamic rendering -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 3 - STUDENT MANAGEMENT -->
        <div id="tab-students" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- Add Single Student -->
                <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                    <h3 class="font-bold text-lg mb-4 text-slate-800">เพิ่มนักเรียนรายบุคคล</h3>
                    <form onsubmit="addSingleStudent(event)" class="space-y-3">
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">ห้องเรียน</label>
                            <select id="single-std-class" required class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm">
                                <option value="">-- เลือกห้องเรียน --</option>
                            </select>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">เลขที่</label>
                                <input type="number" id="single-std-no" required placeholder="1" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสนักเรียน</label>
                                <input type="text" id="single-std-id" required placeholder="10001" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm">
                            </div>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อ - นามสกุล</label>
                            <input type="text" id="single-std-name" required placeholder="นายสมชาย ใจดี" class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm">
                        </div>
                        <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 active:scale-95 text-white font-semibold py-2.5 rounded-xl transition shadow">
                            <i class="fa-solid fa-user-plus mr-1"></i> บันทึกนักเรียน
                        </button>
                    </form>
                </div>

                <!-- Bulk Import from Excel/Text -->
                <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                    <h3 class="font-bold text-lg mb-2 text-slate-800">นำเข้ารายชื่อหลายคน (Bulk Import)</h3>
                    <p class="text-xs text-slate-500 mb-3">คัดลอกรายชื่อจาก Excel ในรูปแบบ: <code>เลขที่[Tab]รหัส[Tab]ชื่อ-นามสกุล</code></p>
                    <form onsubmit="addBulkStudents(event)" class="space-y-3">
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">ห้องเรียน</label>
                            <select id="bulk-std-class" required class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm">
                                <option value="">-- เลือกห้องเรียน --</option>
                            </select>
                        </div>
                        <div>
                            <textarea id="bulk-std-text" rows="5" placeholder="1	10001	นายสมชาย ใจดี
2	10002	นางสาวสมหญิง มีสุข" required class="w-full bg-slate-50 border border-slate-300 rounded-xl p-2.5 text-sm font-mono"></textarea>
                        </div>
                        <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold py-2.5 rounded-xl transition shadow">
                            <i class="fa-solid fa-file-import mr-1"></i> นำเข้าข้อมูลรายชื่อ
                        </button>
                    </form>
                </div>
            </div>

            <!-- Student List by Class -->
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-2">
                    <h3 class="font-bold text-lg text-slate-800">รายชื่อนักเรียนในระบบ</h3>
                    <select id="filter-std-class" onchange="renderStudentsList()" class="bg-slate-50 border border-slate-300 rounded-xl p-2 text-sm">
                        <option value="ALL">-- แสดงทุกห้องเรียน --</option>
                    </select>
                </div>
                <div class="overflow-x-auto rounded-xl border border-slate-200">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-slate-100 text-slate-600 font-semibold text-xs uppercase">
                            <tr>
                                <th class="p-3 border-b">ห้องเรียน</th>
                                <th class="p-3 border-b">เลขที่</th>
                                <th class="p-3 border-b">รหัสนักเรียน</th>
                                <th class="p-3 border-b">ชื่อ - นามสกุล</th>
                                <th class="p-3 border-b text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="students-table-tbody" class="divide-y divide-slate-100 bg-white">
                            <!-- Dynamic render -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- TEACHER VIEW: TAB 4 - REPORTS -->
        <div id="tab-reports" class="tab-content hidden space-y-6">
            <div class="glass-card rounded-2xl p-6 shadow-sm border border-slate-200">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-6 gap-4">
                    <div>
                        <h3 class="font-bold text-lg text-slate-800">รายงานประวัติการเช็คชื่อย้อนหลัง</h3>
                        <p class="text-xs text-slate-500">ค้นหาประวัติ session การเข้าเรียนที่ผ่านมา</p>
                    </div>
                    <button onclick="exportReportsToCSV()" class="bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-medium px-4 py-2 rounded-xl text-sm transition shadow flex items-center">
                        <i class="fa-solid fa-file-excel mr-2"></i> ส่งออกเป็นไฟล์ CSV (Excel)
                    </button>
                </div>

                <div class="overflow-x-auto rounded-xl border border-slate-200">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-slate-100 text-slate-600 font-semibold text-xs uppercase">
                            <tr>
                                <th class="p-3 border-b">วัน-เวลา</th>
                                <th class="p-3 border-b">ห้องเรียน</th>
                                <th class="p-3 border-b">วิชา</th>
                                <th class="p-3 border-b text-center">มาเรียน</th>
                                <th class="p-3 border-b text-center">ขาดเรียน</th>
                                <th class="p-3 border-b text-center">รายละเอียด</th>
                            </tr>
                        </thead>
                        <tbody id="reports-table-tbody" class="divide-y divide-slate-100 bg-white">
                            <!-- Dynamic render -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

    </main>

    <!-- Application Logic & REST Cloud Sync -->
    <script>
        // Public Cloud Firebase REST Endpoint (No API Key Required for Public Realtime Read/Write)
        const CLOUD_DB_BASE_URL = "https://smart-checkin-default-rtdb.asia-southeast1.firebasedatabase.app/activeSessions";

        let pollingInterval = null;
        let currentSessionId = null;

        // Local State Management
        window.appState = {
            classes: JSON.parse(localStorage.getItem('sc_classes')) || [
                { id: 'c1', name: 'ม.4/1', desc: 'สายวิทยาศาสตร์-คณิตศาสตร์' },
                { id: 'c2', name: 'ม.4/2', desc: 'สายภาษา-สังคม' }
            ],
            students: JSON.parse(localStorage.getItem('sc_students')) || [
                { id: 's1', classId: 'c1', no: 1, stdId: '10001', name: 'นายกิตติพงษ์ วงศ์สว่าง' },
                { id: 's2', classId: 'c1', no: 2, stdId: '10002', name: 'นางสาวจิราพร แสงทอง' },
                { id: 's3', classId: 'c1', no: 3, stdId: '10003', name: 'นายธนกร รัตนเดช' },
                { id: 's4', classId: 'c2', no: 1, stdId: '20001', name: 'นางสาวปรียาพร พรหมมา' }
            ],
            history: JSON.parse(localStorage.getItem('sc_history')) || []
        };

        function saveLocalState() {
            localStorage.setItem('sc_classes', JSON.stringify(window.appState.classes));
            localStorage.setItem('sc_students', JSON.stringify(window.appState.students));
            localStorage.setItem('sc_history', JSON.stringify(window.appState.history));
        }

        // Initialize Router on Load
        window.onload = function() {
            checkUrlSession();
        };

        function checkUrlSession() {
            const urlParams = new URLSearchParams(window.location.search);
            const sessionId = urlParams.get('session');

            if (sessionId) {
                // STUDENT MODE
                document.getElementById('teacher-nav').classList.add('hidden');
                document.getElementById('teacher-nav-mobile').classList.add('hidden');
                document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
                document.getElementById('student-view').classList.remove('hidden');
                initStudentSession(sessionId);
            } else {
                // TEACHER MODE
                document.getElementById('teacher-nav').classList.remove('hidden');
                document.getElementById('student-view').classList.add('hidden');
                renderTeacherDropdowns();
                renderClassesList();
                renderStudentsList();
                renderReportsList();
            }
        }

        window.switchTab = function(tabName) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById(`tab-${tabName}`).classList.remove('hidden');

            document.querySelectorAll('.nav-btn').forEach(btn => {
                btn.classList.remove('text-blue-400');
                btn.classList.add('text-slate-300');
            });
            const activeBtn = document.getElementById(`nav-${tabName}`);
            if (activeBtn) {
                activeBtn.classList.remove('text-slate-300');
                activeBtn.classList.add('text-blue-400');
            }
        };

        // --- STUDENT CLOUD LOGIC ---
        async function initStudentSession(sessionId) {
            currentSessionId = sessionId;

            // Start polling for active session updates
            fetchStudentSessionData();
            if (pollingInterval) clearInterval(pollingInterval);
            pollingInterval = setInterval(fetchStudentSessionData, 2000);
        }

        async function fetchStudentSessionData() {
            if (!currentSessionId) return;

            try {
                const response = await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`);
                const data = await response.json();

                if (data && data.active) {
                    document.getElementById('student-class-title').innerText = `${data.className}`;
                    document.getElementById('student-session-info').innerText = `วิชา: ${data.subjectName} | ${data.dateStr}`;

                    const select = document.getElementById('student-dropdown');
                    const currentSelected = select.value;
                    
                    // Maintain selection list
                    if (select.children.length <= 1) {
                        select.innerHTML = '<option value="">-- กรุณาเลือกรายชื่อ --</option>';
                        data.students.forEach(std => {
                            const opt = document.createElement('option');
                            opt.value = std.stdId;
                            opt.textContent = `เลขที่ ${std.no} - ${std.name}`;
                            select.appendChild(opt);
                        });
                    }

                    // Check selected student status change
                    if (currentSelected) {
                        const myRecord = data.students.find(s => s.stdId === currentSelected);
                        if (myRecord) {
                            if (myRecord.status === 'PRESENT') {
                                document.getElementById('student-step-select').classList.add('hidden');
                                document.getElementById('student-step-otp').classList.add('hidden');
                                document.getElementById('student-step-success').classList.remove('hidden');
                            }
                        }
                    }
                } else {
                    document.getElementById('student-class-title').innerText = "ไม่พบข้อมูลคาบเรียน";
                    document.getElementById('student-session-info').innerText = "คาบเรียนนี้ปิดลงแล้ว หรือลิงก์ไม่ถูกต้อง";
                }
            } catch (err) {
                console.error("Cloud fetch error:", err);
            }
        }

        window.requestStudentOTP = async function() {
            const stdId = document.getElementById('student-dropdown').value;
            if (!stdId) {
                Swal.fire({ icon: 'warning', title: 'กรุณาเลือกรายชื่อ', text: 'โปรดเลือกชื่อของคุณก่อนกดขอ OTP ครับ', confirmColor: '#0284c7' });
                return;
            }

            const otpCode = Math.floor(100000 + Math.random() * 900000).toString();

            try {
                // Fetch current state
                const response = await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`);
                const sessionData = await response.json();

                if (sessionData && sessionData.active) {
                    const updatedStudents = sessionData.students.map(s => {
                        if (s.stdId === stdId) {
                            return { ...s, status: 'PENDING', otp: otpCode, requestTime: new Date().toLocaleTimeString('th-TH') };
                        }
                        return s;
                    });

                    // Update Cloud Database via REST PATCH
                    await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`, {
                        method: 'PATCH',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ students: updatedStudents })
                    });

                    document.getElementById('student-step-select').classList.add('hidden');
                    document.getElementById('student-step-otp').classList.remove('hidden');
                    document.getElementById('display-otp-code').innerText = otpCode;

                    Swal.fire({
                        icon: 'success',
                        title: 'สร้างรหัส OTP สำเร็จ!',
                        text: `รหัสของคุณคือ ${otpCode} โปรดแจ้งคุณครูเพื่อยืนยัน`,
                        confirmColor: '#0284c7'
                    });
                }
            } catch (err) {
                console.error("Error requesting OTP:", err);
                Swal.fire({ icon: 'error', title: 'เกิดข้อผิดพลาด', text: 'ไม่สามารถส่งรหัสไปยัง Cloud ได้', confirmColor: '#0284c7' });
            }
        };

        // --- TEACHER CLOUD LOGIC ---
        window.toggleClassSession = async function() {
            const btn = document.getElementById('btn-toggle-session');
            const classSelect = document.getElementById('teacher-class-select');
            const subjectInput = document.getElementById('teacher-subject-input');

            if (!currentSessionId) {
                // START SESSION
                if (!classSelect.value) {
                    Swal.fire({ icon: 'warning', title: 'กรุณาเลือกห้องเรียน', text: 'เลือกห้องเรียนก่อนเริ่มคาบเรียนนะครับ', confirmColor: '#0284c7' });
                    return;
                }
                const selectedClass = window.appState.classes.find(c => c.id === classSelect.value);
                const subjectName = subjectInput.value.trim() || 'วิชาทั่วไป';

                const classStudents = window.appState.students
                    .filter(s => s.classId === selectedClass.id)
                    .map(s => ({
                        stdId: s.stdId,
                        no: s.no,
                        name: s.name,
                        status: 'ABSENT',
                        otp: null
                    }));

                if (classStudents.length === 0) {
                    Swal.fire({ icon: 'info', title: 'ห้องเรียนว่างเปล่า', text: 'ยังไม่มีรายชื่อนักเรียนในห้องเรียนนี้ กรุณาเพิ่มนักเรียนก่อน', confirmColor: '#0284c7' });
                    return;
                }

                currentSessionId = 'S_' + Date.now();
                const sessionData = {
                    sessionId: currentSessionId,
                    classId: selectedClass.id,
                    className: selectedClass.name,
                    subjectName: subjectName,
                    dateStr: new Date().toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric', hour: '2-digit', minute: '2-digit' }),
                    students: classStudents,
                    active: true
                };

                try {
                    // Push Active Session to Public Cloud DB
                    await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`, {
                        method: 'PUT',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(sessionData)
                    });

                    // Start Polling
                    if (pollingInterval) clearInterval(pollingInterval);
                    pollingInterval = setInterval(fetchTeacherLiveSession, 2000);
                    fetchTeacherLiveSession();

                    // Update UI
                    document.getElementById('active-session-container').classList.remove('hidden');
                    document.getElementById('live-class-name').innerText = selectedClass.name;
                    document.getElementById('live-subject-name').innerText = `วิชา: ${subjectName}`;
                    
                    const studentUrl = `${window.location.origin}${window.location.pathname}?session=${currentSessionId}`;
                    document.getElementById('session-link-input').value = studentUrl;
                    
                    const qrcodeDiv = document.getElementById('qrcode');
                    qrcodeDiv.innerHTML = '';
                    new QRCode(qrcodeDiv, {
                        text: studentUrl,
                        width: 160,
                        height: 160,
                        colorDark : "#0284c7",
                        colorLight : "#ffffff"
                    });

                    btn.className = "w-full bg-red-600 hover:bg-red-700 active:scale-95 text-white font-semibold p-2.5 rounded-xl shadow-md transition flex items-center justify-center space-x-2";
                    btn.innerHTML = `<i class="fa-solid fa-stop"></i> <span>🔴 ปิดคาบเรียนออนไลน์</span>`;

                    Swal.fire({ icon: 'success', title: 'เปิดคาบเรียนสำเร็จ!', text: 'ข้อมูลถูก Push ขึ้น Cloud Online DB เรียบร้อยแล้ว', confirmColor: '#0284c7', timer: 1800 });
                } catch (err) {
                    console.error("Error creating session:", err);
                    Swal.fire({ icon: 'error', title: 'การเชื่อมต่อขัดข้อง', text: 'ไม่สามารถสร้างคาบเรียนบน Cloud ได้', confirmColor: '#0284c7' });
                }
            } else {
                // STOP SESSION
                try {
                    // Mark session in Cloud as inactive
                    await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`, {
                        method: 'PATCH',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ active: false })
                    });

                    // Save History
                    const res = await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`);
                    const finalData = await res.json();
                    if (finalData) {
                        window.appState.history.push(finalData);
                        saveLocalState();
                        renderReportsList();
                    }

                    if (pollingInterval) clearInterval(pollingInterval);
                    currentSessionId = null;

                    document.getElementById('active-session-container').classList.add('hidden');
                    btn.className = "w-full bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-semibold p-2.5 rounded-xl shadow-md transition flex items-center justify-center space-x-2";
                    btn.innerHTML = `<i class="fa-solid fa-play"></i> <span>🟢 เริ่มเปิดคาบเรียนออนไลน์ (Cloud Sync)</span>`;

                    Swal.fire({ icon: 'info', title: 'ปิดคาบเรียนเรียบร้อย', text: 'บันทึกรายงานเข้าสู่ระบบแล้ว', confirmColor: '#0284c7' });
                } catch (err) {
                    console.error("Error stopping session:", err);
                }
            }
        };

        async function fetchTeacherLiveSession() {
            if (!currentSessionId) return;

            try {
                const response = await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`);
                const session = await response.json();

                if (session && session.students) {
                    renderLiveStudentsTable(session.students);
                }
            } catch (err) {
                console.error("Error fetching live session:", err);
            }
        }

        function renderLiveStudentsTable(students) {
            const tbody = document.getElementById('live-students-tbody');
            tbody.innerHTML = '';

            let cntAbsent = 0, cntPending = 0, cntPresent = 0;

            students.sort((a, b) => a.no - b.no).forEach(std => {
                const tr = document.createElement('tr');
                
                let statusBadge = '';
                if (std.status === 'ABSENT') {
                    cntAbsent++;
                    statusBadge = `<span class="bg-red-100 text-red-700 text-xs px-2.5 py-1 rounded-full font-medium">🔴 ยังไม่เช็ค</span>`;
                } else if (std.status === 'PENDING') {
                    cntPending++;
                    statusBadge = `<span class="bg-amber-100 text-amber-800 text-xs px-2.5 py-1 rounded-full font-medium animate-pulse">🟡 ได้รับ OTP แล้ว</span>`;
                } else if (std.status === 'PRESENT') {
                    cntPresent++;
                    statusBadge = `<span class="bg-emerald-100 text-emerald-800 text-xs px-2.5 py-1 rounded-full font-medium">🟢 เข้าเรียนแล้ว</span>`;
                }

                tr.innerHTML = `
                    <td class="p-3 font-medium text-slate-700">${std.no}</td>
                    <td class="p-3 font-mono text-xs text-slate-500">${std.stdId}</td>
                    <td class="p-3 font-medium text-slate-800">${std.name}</td>
                    <td class="p-3 text-center">${statusBadge}</td>
                    <td class="p-3 text-center font-mono font-bold text-amber-600">${std.otp || '-'}</td>
                    <td class="p-3 text-center">
                        ${std.status !== 'PRESENT' ? `
                            <button onclick="approveStudentPresent('${std.stdId}')" class="bg-emerald-500 hover:bg-emerald-600 text-white text-xs px-3 py-1.5 rounded-lg shadow transition active:scale-95">
                                <i class="fa-solid fa-check mr-1"></i> อนุมัติ
                            </button>
                        ` : `
                            <span class="text-xs text-emerald-600 font-semibold"><i class="fa-solid fa-check-double"></i> สมบูรณ์</span>
                        `}
                    </td>
                `;
                tbody.appendChild(tr);
            });

            document.getElementById('cnt-absent').innerText = cntAbsent;
            document.getElementById('cnt-pending').innerText = cntPending;
            document.getElementById('cnt-present').innerText = cntPresent;
        }

        window.approveStudentPresent = async function(stdId) {
            if (!currentSessionId) return;

            try {
                const res = await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`);
                const session = await res.json();

                if (session) {
                    const updated = session.students.map(s => {
                        if (s.stdId === stdId) {
                            return { ...s, status: 'PRESENT' };
                        }
                        return s;
                    });

                    await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`, {
                        method: 'PATCH',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ students: updated })
                    });

                    fetchTeacherLiveSession();
                }
            } catch (err) {
                console.error("Approve error:", err);
            }
        };

        window.verifyTeacherOTP = async function() {
            const input = document.getElementById('teacher-otp-input');
            const otpVal = input.value.trim();

            if (!otpVal || otpVal.length !== 6) {
                Swal.fire({ icon: 'warning', title: 'โปรดระบุ OTP', text: 'กรอกรหัส OTP 6 หลักให้ครบถ้วน', confirmColor: '#0284c7' });
                return;
            }

            try {
                const res = await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`);
                const session = await res.json();

                if (session) {
                    let matched = false;
                    const updated = session.students.map(s => {
                        if (s.otp === otpVal && s.status !== 'PRESENT') {
                            matched = true;
                            return { ...s, status: 'PRESENT' };
                        }
                        return s;
                    });

                    if (matched) {
                        await fetch(`${CLOUD_DB_BASE_URL}/${currentSessionId}.json`, {
                            method: 'PATCH',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify({ students: updated })
                        });

                        input.value = '';
                        fetchTeacherLiveSession();
                        Swal.fire({ icon: 'success', title: 'อนุมัติเรียบร้อย!', text: 'ยืนยันการเข้าเรียนของนักเรียนสำเร็จ', confirmColor: '#0284c7', timer: 1500 });
                    } else {
                        Swal.fire({ icon: 'error', title: 'ไม่พบ OTP', text: 'รหัส OTP ไม่ถูกต้องหรือถูกใช้งานไปแล้ว', confirmColor: '#0284c7' });
                    }
                }
            } catch (err) {
                console.error("OTP verification error:", err);
            }
        };

        // --- MANAGEMENT METHODS ---
        function renderTeacherDropdowns() {
            const classSelects = [
                document.getElementById('teacher-class-select'),
                document.getElementById('single-std-class'),
                document.getElementById('bulk-std-class'),
                document.getElementById('filter-std-class')
            ];

            classSelects.forEach(select => {
                if (!select) return;
                const isFilter = select.id === 'filter-std-class';
                select.innerHTML = isFilter ? '<option value="ALL">-- แสดงทุกห้องเรียน --</option>' : '<option value="">-- เลือกห้องเรียน --</option>';

                window.appState.classes.forEach(c => {
                    const opt = document.createElement('option');
                    opt.value = c.id;
                    opt.textContent = c.name;
                    select.appendChild(opt);
                });
            });
        }

        window.addClass = function(e) {
            e.preventDefault();
            const nameInput = document.getElementById('input-class-name');
            const descInput = document.getElementById('input-class-desc');

            const newClass = {
                id: 'c_' + Date.now(),
                name: nameInput.value.trim(),
                desc: descInput.value.trim()
            };

            window.appState.classes.push(newClass);
            saveLocalState();
            nameInput.value = '';
            descInput.value = '';
            renderTeacherDropdowns();
            renderClassesList();
            Swal.fire({ icon: 'success', title: 'เพิ่มห้องเรียนแล้ว', confirmColor: '#0284c7', timer: 1200 });
        };

        function renderClassesList() {
            const tbody = document.getElementById('classes-table-tbody');
            tbody.innerHTML = '';

            window.appState.classes.forEach((c, index) => {
                const studentCount = window.appState.students.filter(s => s.classId === c.id).length;
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 font-medium text-slate-500">${index + 1}</td>
                    <td class="p-3 font-bold text-slate-800">${c.name}</td>
                    <td class="p-3 text-slate-600">${c.desc || '-'}</td>
                    <td class="p-3 text-center"><span class="bg-blue-100 text-blue-700 font-semibold px-2.5 py-0.5 rounded-full text-xs">${studentCount} คน</span></td>
                    <td class="p-3 text-center">
                        <button onclick="removeClass('${c.id}')" class="text-red-500 hover:text-red-700 p-1 rounded transition">
                            <i class="fa-solid fa-trash"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        window.removeClass = function(classId) {
            window.appState.classes = window.appState.classes.filter(c => c.id !== classId);
            window.appState.students = window.appState.students.filter(s => s.classId !== classId);
            saveLocalState();
            renderTeacherDropdowns();
            renderClassesList();
            renderStudentsList();
        };

        window.addSingleStudent = function(e) {
            e.preventDefault();
            const classId = document.getElementById('single-std-class').value;
            const no = parseInt(document.getElementById('single-std-no').value);
            const stdId = document.getElementById('single-std-id').value.trim();
            const name = document.getElementById('single-std-name').value.trim();

            window.appState.students.push({
                id: 's_' + Date.now(),
                classId,
                no,
                stdId,
                name
            });

            saveLocalState();
            document.getElementById('single-std-no').value = '';
            document.getElementById('single-std-id').value = '';
            document.getElementById('single-std-name').value = '';

            renderStudentsList();
            renderClassesList();
            Swal.fire({ icon: 'success', title: 'เพิ่มนักเรียนสำเร็จ', confirmColor: '#0284c7', timer: 1200 });
        };

        window.addBulkStudents = function(e) {
            e.preventDefault();
            const classId = document.getElementById('bulk-std-class').value;
            const rawText = document.getElementById('bulk-std-text').value.trim();

            const lines = rawText.split('\n');
            let addedCount = 0;

            lines.forEach((line) => {
                const parts = line.split('\t').map(p => p.trim());
                if (parts.length >= 3) {
                    window.appState.students.push({
                        id: 's_' + Date.now() + '_' + Math.random(),
                        classId,
                        no: parseInt(parts[0]) || 0,
                        stdId: parts[1],
                        name: parts[2]
                    });
                    addedCount++;
                }
            });

            saveLocalState();
            document.getElementById('bulk-std-text').value = '';
            renderStudentsList();
            renderClassesList();
            Swal.fire({ icon: 'success', title: `นำเข้าสำเร็จ ${addedCount} คน`, confirmColor: '#0284c7' });
        };

        window.renderStudentsList = function() {
            const filterClassId = document.getElementById('filter-std-class')?.value || 'ALL';
            const tbody = document.getElementById('students-table-tbody');
            tbody.innerHTML = '';

            const filtered = filterClassId === 'ALL' 
                ? window.appState.students 
                : window.appState.students.filter(s => s.classId === filterClassId);

            filtered.sort((a, b) => a.no - b.no).forEach(s => {
                const cls = window.appState.classes.find(c => c.id === s.classId);
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 text-slate-600">${cls ? cls.name : '-'}</td>
                    <td class="p-3 font-medium">${s.no}</td>
                    <td class="p-3 font-mono text-xs text-slate-500">${s.stdId}</td>
                    <td class="p-3 font-medium text-slate-800">${s.name}</td>
                    <td class="p-3 text-center">
                        <button onclick="removeStudent('${s.id}')" class="text-red-500 hover:text-red-700 p-1 transition">
                            <i class="fa-solid fa-trash"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        };

        window.removeStudent = function(studentId) {
            window.appState.students = window.appState.students.filter(s => s.id !== studentId);
            saveLocalState();
            renderStudentsList();
            renderClassesList();
        };

        function renderReportsList() {
            const tbody = document.getElementById('reports-table-tbody');
            tbody.innerHTML = '';

            if (window.appState.history.length === 0) {
                tbody.innerHTML = `<tr><td colspan="6" class="text-center py-6 text-slate-400">ยังไม่มีประวัติการเช็คชื่อย้อนหลัง</td></tr>`;
                return;
            }

            window.appState.history.forEach(session => {
                const presentCnt = session.students.filter(s => s.status === 'PRESENT').length;
                const absentCnt = session.students.filter(s => s.status !== 'PRESENT').length;

                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 text-slate-600 font-medium">${session.dateStr}</td>
                    <td class="p-3 font-bold text-slate-800">${session.className}</td>
                    <td class="p-3 text-slate-600">${session.subjectName}</td>
                    <td class="p-3 text-center font-semibold text-emerald-600">${presentCnt} คน</td>
                    <td class="p-3 text-center font-semibold text-red-500">${absentCnt} คน</td>
                    <td class="p-3 text-center">
                        <span class="text-xs bg-slate-100 text-slate-600 px-2 py-1 rounded font-medium">เสร็จสิ้น</span>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        window.exportReportsToCSV = function() {
            if (window.appState.history.length === 0) {
                Swal.fire({ icon: 'warning', title: 'ไม่มีข้อมูล', text: 'ยังไม่มีรายงานย้อนหลังสำหรับส่งออก', confirmColor: '#0284c7' });
                return;
            }

            let csvContent = "\uFEFF";
            csvContent += "วัน-เวลา,ห้องเรียน,ชื่อวิชา,รหัสนักเรียน,ชื่อ-นามสกุล,สถานะการเช็คชื่อ\n";

            window.appState.history.forEach(session => {
                session.students.forEach(std => {
                    const statusText = std.status === 'PRESENT' ? 'มาเรียน' : 'ขาดเรียน';
                    csvContent += `"${session.dateStr}","${session.className}","${session.subjectName}","${std.stdId}","${std.name}","${statusText}"\n`;
                });
            });

            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `Smart_CheckIn_Report_${Date.now()}.csv`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
        };

        window.copySessionLink = function() {
            const copyText = document.getElementById("session-link-input");
            copyText.select();
            document.execCommand("copy");
            Swal.fire({ icon: 'success', title: 'คัดลอกลิงก์สำเร็จ!', text: 'สามารถนำลิงก์ไปส่งให้ นักเรียน ได้ทันที', confirmColor: '#0284c7', timer: 1500 });
        };
    </script>
</body>
</html>
