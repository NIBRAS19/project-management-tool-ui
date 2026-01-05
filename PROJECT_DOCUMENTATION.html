const fs = require('fs');
const path = require('path');

const filePath = path.join(__dirname, 'PROJECT_DOCUMENTATION.html');

try {
    const content = fs.readFileSync(filePath, 'utf8');

    // Check if already HTML wrapped (simple check)
    if (content.trim().startsWith('<!DOCTYPE html>')) {
        console.log('File already appears to be HTML. Aborting to prevent double-wrapping.');
        process.exit(0);
    }

    const jsonContent = JSON.stringify(content);

    const html = `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Project Management System - Documentation</title>
    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
    
    <!-- Syntax Highlighting -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/atom-one-dark.min.css">
    
    <!-- Markdown Parser -->
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
    
    <style>
        :root {
            --bg-color: #0f172a;
            --surface-color: #1e293b;
            --text-primary: #f8fafc;
            --text-secondary: #cbd5e1;
            --accent-color: #38bdf8;
            --border-color: #334155;
            --code-bg: #1e293b;
            --header-border: #334155;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-primary);
            font-family: 'Inter', system-ui, -apple-system, sans-serif;
            line-height: 1.7;
            margin: 0;
            padding: 0;
        }

        .container {
            max-width: 1024px;
            margin: 0 auto;
            padding: 40px 24px;
        }

        /* Typography */
        h1, h2, h3, h4, h5, h6 {
            color: #fff;
            margin-top: 2rem;
            margin-bottom: 1rem;
            font-weight: 700;
            line-height: 1.3;
        }

        h1 {
            font-size: 2.5rem;
            background: linear-gradient(to right, #38bdf8, #818cf8);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin-bottom: 2rem;
            padding-bottom: 1rem;
            border-bottom: 1px solid var(--border-color);
        }

        h2 {
            font-size: 1.8rem;
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 0.5rem;
            color: #e2e8f0;
        }

        h3 {
            font-size: 1.4rem;
            color: #f1f5f9;
        }

        p {
            margin-bottom: 1.2rem;
            color: var(--text-secondary);
        }

        a {
            color: var(--accent-color);
            text-decoration: none;
            transition: color 0.2s;
            border-bottom: 1px dashed transparent;
        }

        a:hover {
            color: #7dd3fc;
            border-bottom-color: #7dd3fc;
        }

        /* Lists */
        ul, ol {
            padding-left: 1.5rem;
            margin-bottom: 1.5rem;
            color: var(--text-secondary);
        }

        li {
            margin-bottom: 0.5rem;
        }

        /* Code Blocks */
        pre {
            background-color: var(--code-bg);
            padding: 1.5rem;
            border-radius: 12px;
            overflow-x: auto;
            margin: 1.5rem 0;
            border: 1px solid var(--border-color);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }

        code {
            font-family: 'JetBrains Mono', monospace;
            font-size: 0.9em;
        }

        /* Inline code */
        p code, li code, td code, th code {
            background-color: rgba(56, 189, 248, 0.1);
            color: #7dd3fc;
            padding: 0.2rem 0.4rem;
            border-radius: 6px;
        }

        /* Tables */
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 1.5rem 0;
            background-color: var(--surface-color);
            border-radius: 8px;
            overflow: hidden;
            border: 1px solid var(--border-color);
        }

        th, td {
            padding: 12px 16px;
            text-align: left;
            border-bottom: 1px solid var(--border-color);
        }

        th {
            background-color: rgba(56, 189, 248, 0.05);
            color: #fff;
            font-weight: 600;
        }

        tr:last-child td {
            border-bottom: none;
        }

        /* Blockquotes */
        blockquote {
            border-left: 4px solid var(--accent-color);
            background: rgba(56, 189, 248, 0.05);
            margin: 1.5rem 0;
            padding: 1rem 1.5rem;
            border-radius: 0 8px 8px 0;
            color: #e2e8f0;
            font-style: italic;
        }

        /* Horizontal Rule */
        hr {
            border: 0;
            height: 1px;
            background: var(--border-color);
            margin: 3rem 0;
        }

        /* Images */
        img {
            max-width: 100%;
            height: auto;
            border-radius: 8px;
            border: 1px solid var(--border-color);
        }
    </style>
</head>
<body>
    <div class="container">
        <div id="content"></div>
    </div>

    <script>
        const markdownContent = ${jsonContent};
        
        // Configure marked to use highlight.js
        marked.setOptions({
            highlight: function(code, lang) {
                const language = hljs.getLanguage(lang) ? lang : 'plaintext';
                return hljs.highlight(code, { language }).value;
            },
            langPrefix: 'hljs language-',
        });

        document.getElementById('content').innerHTML = marked.parse(markdownContent);
        
        // Ensure all code blocks are highlighted (backup if marked option fails)
        document.querySelectorAll('pre code').forEach((block) => {
            hljs.highlightElement(block);
        });
    </script>
</body>
</html>`;

    fs.writeFileSync(filePath, html);
    console.log('Successfully wrapped PROJECT_DOCUMENTATION.html with HTML and CSS.');

} catch (err) {
    console.error('Error processing file:', err);
    process.exit(1);
}
