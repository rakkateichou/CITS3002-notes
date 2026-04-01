- [[Week 1]]
- [[Week 2]]
- [[Week 3]]
- [[Week 4]]
- [[Week 5]]
- [[Practice Mid-Semester Test 1]]
- [[Practice Mid-Semester Test 2]]
- [[Practice Mid-Semester Test 3]]

```dataviewjs
const { execSync, exec } = require('child_process');
const fs = require('fs');
const path = require('path');
const os = require('os');

// --- Configuration ---
const quartzDir = os.homedir() + '/Developer/quartz/CITS3002-notes';
const contentDir = path.join(quartzDir, 'content');
const vaultBase = '/Users/rakkate/Library/Mobile Documents/iCloud~md~obsidian/Documents/MyVault/University/Semester 4/CITS3002 - Computer Networks/';

const customEnv = Object.assign({}, process.env);
customEnv.PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin";

// --- UI Elements ---
const button = dv.container.createEl('button', { text: '🚀 Auto-Sync All Notes' });
button.addClass('mod-cta');
button.style.width = "100%";

const outputDiv = dv.container.createEl('pre', { text: 'Ready to scan and sync.' });
outputDiv.style.whiteSpace = "pre-wrap";
outputDiv.style.background = "var(--background-secondary)";
outputDiv.style.padding = "10px";
outputDiv.style.fontSize = "0.85em";

button.onclick = async () => {
    button.disabled = true;
    button.innerText = 'Processing...';
    outputDiv.innerText = "Step 1: Cleaning and flattening files...\n";

    try {
        // 1. Ensure content directory exists and is clean-ish 
        // (We don't delete everything to avoid breaking Quartz metadata, but we ensure it exists)
        if (!fs.existsSync(contentDir)) fs.mkdirSync(contentDir, { recursive: true });

        // 2. Find all .md files recursively and copy them to the flat content root
        // The -exec cp {} shell command handles spaces correctly
        const copyCmd = `find "${vaultBase}" -name "*.md" -exec cp "{}" "${contentDir}/" \\;`;
        execSync(copyCmd, { shell: '/bin/zsh' });

        // 3. Generate the index.md based on files actually copied
        outputDiv.innerText += "Step 2: Generating index.md...\n";
        
        const files = fs.readdirSync(contentDir)
            .filter(f => f.endsWith('.md') && f.toLowerCase() !== 'index.md')
            .sort();

        const indexContent = [
            "---",
            "title: CITS3002 - Computer Networks",
            "---",
            "# Index",
            "",
            files.map(f => `- [[${f.replace('.md', '')}]]`).join('\n')
        ].join('\n');

        fs.writeFileSync(path.join(contentDir, 'index.md'), indexContent);

        // 4. Run Quartz Sync
        outputDiv.innerText += "Step 3: Running Quartz Sync...\n\n";
        
        exec('/opt/homebrew/bin/npx quartz sync --no-pull', { 
            cwd: quartzDir, 
            env: customEnv, 
            shell: '/bin/zsh' 
        }, (error, stdout, stderr) => {
            if (error) {
                outputDiv.innerText += "❌ ERROR:\n" + error.message;
            } else {
                outputDiv.innerText += stdout + "\n✅ Sync Complete!";
            }
            button.disabled = false;
            button.innerText = '🚀 Auto-Sync All Notes';
        });

    } catch (err) {
        outputDiv.innerText += "❌ SCRIPT CRASHED:\n" + err.message;
        button.disabled = false;
        button.innerText = '🚀 Auto-Sync All Notes';
    }
};
```