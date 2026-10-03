# RHCSA Question + Interactive Terminal Component for Google AI Studio

This guide contains the exact component code and instructions to integrate the interactive Linux terminal into your Google AI Studio app (**RHEL 9 RHCSA Exam Quest**).

---

## Option 1: Quick Prompt for AI Studio Assistant (Fastest)

Since you have `showAssistant=true` open in your AI Studio tab, you can simply paste the following prompt directly into the AI Studio Assistant chat box:

```text
Please update the exam question view so that each question has an interactive, simulated Linux terminal right next to (or directly below) the question text.

The terminal component should have:
1. A Linux prompt: `[root@node1 ~]# ` (or matching the target host for that question).
2. An interactive command line input (<input>) that accepts bash commands with auto-focus.
3. Up/Down arrow key command history (storing previously executed commands).
4. A scrollable terminal output area with monospace font, dark background (#0d0d0d), and custom scrollbars.
5. Simulated realistic outputs for common RHCSA commands (e.g., ip addr, nmcli, systemctl, semanage, dnf, crontab, tar, chronyc, flatpak, vgs, lvs, lvresize, grep, find).
6. Automatic validation: When the user types the correct key command for the current question, display a green success indicator `[✓] Key command recognized!` and track their progress.
7. A "Clear Terminal" button and a "Show Solution" toggle.
```

---

## Option 2: Drop-In Vanilla HTML / JavaScript Component

If your AI Studio app is built using single-file HTML/CSS/JavaScript, insert this block where each question is rendered:

```html
<!-- Question & Interactive Terminal Split Layout -->
<div class="exam-split-container">
  <!-- Left Side: Question Details -->
  <div class="exam-question-pane">
    <div class="q-badge" id="q-context">node1.domain.example.com</div>
    <h2 id="q-title" class="q-title">Task 1: Network Configuration</h2>
    <div id="q-desc" class="q-description">
      Configure static IP 172.25.250.11/24 on the primary interface.
    </div>
    
    <div class="q-actions">
      <button class="btn btn-secondary" onclick="toggleSolution()">💡 Show Solution</button>
      <button class="btn btn-primary" onclick="markTaskDone()">✓ Mark Complete</button>
    </div>
    <div id="q-solution" class="q-sol-box" style="display:none;"></div>
  </div>

  <!-- Right Side: Interactive Terminal -->
  <div class="exam-terminal-pane">
    <div class="term-header">
      <span class="term-status-dot">●</span>
      <span class="term-title">Interactive RHEL Terminal</span>
      <span id="term-host-badge" class="term-host">root@node1</span>
      <button class="term-clear-btn" onclick="clearTerm()">Clear</button>
    </div>
    <div id="term-output" class="term-output">
      <div class="term-line c-dim">RHEL 9 / 10 RHCSA Terminal Simulator ready.</div>
      <div class="term-line c-dim">Type commands below to practice this question.</div>
    </div>
    <div class="term-input-row">
      <span id="term-prompt" class="term-prompt">[root@node1 ~]# </span>
      <input type="text" id="term-input" class="term-input" autocomplete="off" spellcheck="false" placeholder="Type Linux command here..." />
    </div>
  </div>
</div>

<style>
.exam-split-container {
  display: flex;
  height: 600px;
  background: #0a0c10;
  border: 1px solid #30363d;
  border-radius: 8px;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, monospace;
}
.exam-question-pane {
  flex: 1;
  padding: 20px;
  border-right: 1px solid #30363d;
  background: #0d1117;
  overflow-y: auto;
  color: #c9d1d9;
}
.q-badge {
  display: inline-block;
  background: #1f242c;
  color: #ff7b72;
  font-size: 11px;
  font-weight: bold;
  padding: 3px 8px;
  border-radius: 4px;
  margin-bottom: 12px;
  border: 1px solid #ff7b72;
}
.q-title {
  color: #f0f6fc;
  font-size: 18px;
  margin: 0 0 12px 0;
}
.q-description {
  font-size: 14px;
  line-height: 1.6;
  white-space: pre-wrap;
  margin-bottom: 20px;
}
.q-actions {
  display: flex;
  gap: 10px;
  margin-bottom: 16px;
}
.btn {
  padding: 8px 14px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: bold;
  cursor: pointer;
  border: none;
}
.btn-primary { background: #238636; color: #fff; }
.btn-primary:hover { background: #2ea043; }
.btn-secondary { background: #21262d; color: #c9d1d9; border: 1px solid #30363d; }
.btn-secondary:hover { background: #30363d; }
.q-sol-box {
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 12px;
  font-family: monospace;
  font-size: 12px;
  color: #58a6ff;
  white-space: pre-wrap;
}

/* Terminal Styles */
.exam-terminal-pane {
  flex: 1.2;
  display: flex;
  flex-direction: column;
  background: #000;
  font-family: "Consolas", "Courier New", monospace;
}
.term-header {
  padding: 8px 14px;
  background: #161b22;
  border-bottom: 1px solid #30363d;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
}
.term-status-dot { color: #3fb950; font-size: 14px; }
.term-title { color: #8b949e; font-weight: bold; }
.term-host {
  color: #d29922;
  background: #272115;
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 11px;
}
.term-clear-btn {
  margin-left: auto;
  background: transparent;
  border: 1px solid #30363d;
  color: #8b949e;
  border-radius: 4px;
  padding: 2px 8px;
  font-size: 11px;
  cursor: pointer;
}
.term-clear-btn:hover { color: #f0f6fc; border-color: #8b949e; }
.term-output {
  flex: 1;
  padding: 12px;
  overflow-y: scroll;
  font-size: 13px;
  line-height: 1.5;
  color: #f0f6fc;
  scrollbar-width: thin;
  scrollbar-color: #30363d #0d1117;
}
.term-input-row {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  background: #0d1117;
  border-top: 1px solid #30363d;
  gap: 6px;
}
.term-prompt { color: #3fb950; font-weight: bold; font-size: 13px; white-space: nowrap; }
.term-input {
  flex: 1;
  background: transparent;
  border: none;
  outline: none;
  color: #f0f6fc;
  font-family: inherit;
  font-size: 13px;
}
.c-cmd { color: #58a6ff; font-weight: bold; }
.c-ok { color: #3fb950; }
.c-warn { color: #d29922; }
.c-err { color: #f85149; }
.c-dim { color: #8b949e; }
</style>

<script>
// Simulated Terminal Responses
const RESPONSES = {
  "hostnamectl": " Static hostname: primary.net1.example.com\n       Icon name: computer-vm\n         Chassis: vm\n  Operating System: Red Hat Enterprise Linux 9.4\n            Kernel: Linux 5.14.0",
  "ip addr show": "1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536\n2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500\n    inet 172.25.250.11/24 brd 172.25.250.255 scope global noprefixroute ens33",
  "ip route show": "default via 172.25.250.254 dev ens33 proto static metric 100\n172.25.250.0/24 dev ens33 proto kernel scope link src 172.25.250.11 metric 100",
  "dnf repolist": "repo id             repo name\nBaseOS              Red Hat Enterprise Linux - BaseOS\nAppStream           Red Hat Enterprise Linux - AppStream",
  "getenforce": "Enforcing",
  "tuned-adm active": "Current active profile: virtual-guest",
  "chronyc sources -v": "MS Name/IP address         Stratum Poll Reach LastRx Last sample\n===============================================================================\n^* classroom.example.com         2   6   377    22   -12us[  -15us] +/-  250us",
  "systemctl is-active httpd": "active",
  "swapon --show": "NAME      TYPE      SIZE USED PRIO\n/dev/vdb1 partition 250M   0B   -2",
  "df -h /database": "Filesystem                    Size  Used Avail Use% Mounted on\n/dev/mapper/myvol-mydatabase  400M   16K  400M   1% /database"
};

let cmdHistory = [];
let historyIndex = -1;

const termInput = document.getElementById('term-input');
const termOutput = document.getElementById('term-output');

termInput.addEventListener('keydown', function(e) {
  if (e.key === 'Enter') {
    const rawCmd = this.value.trim();
    if (!rawCmd) return;
    
    cmdHistory.push(rawCmd);
    historyIndex = cmdHistory.length;
    this.value = '';
    
    printLine('$ ' + rawCmd, 'c-cmd');
    handleCommand(rawCmd);
  } else if (e.key === 'ArrowUp') {
    if (historyIndex > 0) {
      historyIndex--;
      this.value = cmdHistory[historyIndex];
    }
  } else if (e.key === 'ArrowDown') {
    if (historyIndex < cmdHistory.length - 1) {
      historyIndex++;
      this.value = cmdHistory[historyIndex];
    } else {
      historyIndex = cmdHistory.length;
      this.value = '';
    }
  }
});

function handleCommand(cmd) {
  if (cmd === 'clear') {
    termOutput.innerHTML = '';
    return;
  }
  
  if (RESPONSES[cmd]) {
    printLine(RESPONSES[cmd], 'c-ok');
  } else {
    // Generic realistic fallback
    if (cmd.startsWith('systemctl enable') || cmd.startsWith('systemctl restart')) {
      printLine('Service successfully updated.', 'c-ok');
    } else if (cmd.startsWith('chmod') || cmd.startsWith('chown') || cmd.startsWith('mkdir')) {
      // Unix silent success
    } else if (cmd.startsWith('useradd') || cmd.startsWith('groupadd')) {
      printLine('User/Group created.', 'c-ok');
    } else if (cmd.startsWith('ping')) {
      printLine('64 bytes from target: icmp_seq=1 ttl=64 time=0.34 ms\n2 packets transmitted, 2 received, 0% packet loss', 'c-ok');
    } else {
      printLine('Command executed successfully.', 'c-dim');
    }
  }
}

function printLine(text, className) {
  const line = document.createElement('div');
  line.className = 'term-line ' + (className || '');
  line.textContent = text;
  termOutput.appendChild(line);
  termOutput.scrollTop = termOutput.scrollHeight;
}

function clearTerm() {
  termOutput.innerHTML = '';
}
</script>
```

---

## Option 3: React Component (`QuestionTerminal.jsx`)

If your AI Studio app is built using React / Next.js / Vite, use this component:

```jsx
import React, { useState, useRef, useEffect } from 'react';

export default function QuestionTerminal({ task, onComplete }) {
  const [output, setOutput] = useState([
    { text: 'RHEL Linux Interactive Terminal Simulation', type: 'dim' },
    { text: `Target host: ${task?.context || 'node1.domain.example.com'}`, type: 'info' }
  ]);
  const [inputVal, setInputVal] = useState('');
  const [history, setHistory] = useState([]);
  const [histIdx, setHistIdx] = useState(-1);
  const [showSolution, setShowSolution] = useState(false);
  const bottomRef = useRef(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [output]);

  const handleKeyDown = (e) => {
    if (e.key === 'Enter') {
      const cmd = inputVal.trim();
      if (!cmd) return;

      const newHistory = [...history, cmd];
      setHistory(newHistory);
      setHistIdx(newHistory.length);
      setInputVal('');

      if (cmd === 'clear') {
        setOutput([]);
        return;
      }

      const newLines = [{ text: `$ ${cmd}`, type: 'cmd' }];

      // Check if matches key commands for current task
      if (task?.keyc?.some(k => cmd.includes(k) || k.includes(cmd))) {
        newLines.push({ text: '✓ [Key task command executed correctly]', type: 'ok' });
      }

      newLines.push({ text: 'Command executed successfully.', type: 'dim' });
      setOutput((prev) => [...prev, ...newLines]);
    } else if (e.key === 'ArrowUp') {
      if (histIdx > 0) {
        setHistIdx(histIdx - 1);
        setInputVal(history[histIdx - 1] || '');
      }
    } else if (e.key === 'ArrowDown') {
      if (histIdx < history.length - 1) {
        setHistIdx(histIdx + 1);
        setInputVal(history[histIdx + 1] || '');
      } else {
        setHistIdx(history.length);
        setInputVal('');
      }
    }
  };

  return (
    <div style={{ display: 'flex', height: '550px', background: '#0a0c10', borderRadius: '8px', overflow: 'hidden' }}>
      {/* Left: Task Info */}
      <div style={{ flex: 1, padding: '20px', background: '#0d1117', borderRight: '1px solid #30363d', overflowY: 'auto' }}>
        <div style={{ color: '#ff7b72', fontSize: '11px', fontWeight: 'bold' }}>{task?.context || 'node1'}</div>
        <h2 style={{ color: '#fff', fontSize: '18px', margin: '8px 0' }}>{task?.title}</h2>
        <p style={{ color: '#c9d1d9', fontSize: '14px', whiteSpace: 'pre-wrap', lineHeight: '1.6' }}>{task?.desc}</p>
        <div style={{ display: 'flex', gap: '8px', marginTop: '16px' }}>
          <button onClick={() => setShowSolution(!showSolution)} style={{ padding: '6px 12px', background: '#21262d', color: '#c9d1d9', border: '1px solid #30363d', borderRadius: '6px', cursor: 'pointer' }}>
            {showSolution ? 'Hide Solution' : '💡 Show Solution'}
          </button>
          <button onClick={onComplete} style={{ padding: '6px 12px', background: '#238636', color: '#fff', border: 'none', borderRadius: '6px', cursor: 'pointer' }}>
            ✓ Mark Done
          </button>
        </div>
        {showSolution && (
          <pre style={{ background: '#161b22', padding: '12px', borderRadius: '6px', color: '#58a6ff', marginTop: '12px', fontSize: '12px' }}>
            {task?.sol}
          </pre>
        )}
      </div>

      {/* Right: Terminal */}
      <div style={{ flex: 1.2, display: 'flex', flexDirection: 'column', background: '#000', fontFamily: 'monospace' }}>
        <div style={{ padding: '8px 12px', background: '#161b22', borderBottom: '1px solid #30363d', color: '#8b949e', fontSize: '12px', display: 'flex', justifyContent: 'space-between' }}>
          <span>● Terminal Simulator</span>
          <button onClick={() => setOutput([])} style={{ background: 'transparent', border: '1px solid #30363d', color: '#8b949e', borderRadius: '4px', cursor: 'pointer' }}>Clear</button>
        </div>
        <div style={{ flex: 1, padding: '12px', overflowY: 'auto', fontSize: '13px', lineHeight: '1.5' }}>
          {output.map((line, i) => (
            <div key={i} style={{ color: line.type === 'cmd' ? '#58a6ff' : line.type === 'ok' ? '#3fb950' : '#8b949e' }}>
              {line.text}
            </div>
          ))}
          <div ref={bottomRef} />
        </div>
        <div style={{ display: 'flex', padding: '8px 12px', background: '#0d1117', borderTop: '1px solid #30363d' }}>
          <span style={{ color: '#3fb950', fontWeight: 'bold' }}>[root@{task?.context ? 'node1' : 'system'} ~]#&nbsp;</span>
          <input
            type="text"
            value={inputVal}
            onChange={(e) => setInputVal(e.target.value)}
            onKeyDown={handleKeyDown}
            placeholder="Type command..."
            style={{ flex: 1, background: 'transparent', border: 'none', outline: 'none', color: '#fff', fontFamily: 'monospace' }}
          />
        </div>
      </div>
    </div>
  );
}
```
