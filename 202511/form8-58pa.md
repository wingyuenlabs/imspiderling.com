---
Title: form8
Description: 
Author: Query Filter
Date: 2025-11-13T21:01:40.000Z
Robots: noindex,nofollow
Template: index
---
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;!DOCTYPE html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
    &lt;meta charset="UTF-8"&gt;
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;
    &lt;title&gt;Offline SQL Formatter&lt;/title&gt;
    &lt;style&gt;
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            text-align: center;
        }
        .editor-container {
            display: flex;
            gap: 20px;
            margin-bottom: 20px;
        }
        @media (max-width: 768px) {
            .editor-container {
                flex-direction: column;
            }
        }
        .editor-panel {
            flex: 1;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #555;
        }
        textarea {
            width: 100%;
            height: 300px;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-family: 'Courier New', monospace;
            font-size: 14px;
            resize: vertical;
        }
        .controls {
            margin: 20px 0;
            text-align: center;
        }
        button {
            background-color: #007cba;
            color: white;
            border: none;
            padding: 10px 20px;
            margin: 0 5px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }
        button:hover {
            background-color: #005a87;
        }
        button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        .options {
            margin: 10px 0;
            text-align: center;
        }
        select {
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            margin: 0 5px;
        }
        .status {
            text-align: center;
            margin: 10px 0;
            color: #666;
            font-style: italic;
        }
    &lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div class="container"&gt;
        &lt;h1&gt;Offline SQL Formatter&lt;/h1&gt;

        &lt;div class="options"&gt;
            &lt;label for="dialect"&gt;SQL Dialect:&lt;/label&gt;
            &lt;select id="dialect"&gt;
                &lt;option value="sql"&gt;Standard SQL&lt;/option&gt;
                &lt;option value="mysql"&gt;MySQL&lt;/option&gt;
                &lt;option value="postgresql"&gt;PostgreSQL&lt;/option&gt;
                &lt;option value="mariadb"&gt;MariaDB&lt;/option&gt;
                &lt;option value="sqlite"&gt;SQLite&lt;/option&gt;
            &lt;/select&gt;

            &lt;label for="indent"&gt;Indent:&lt;/label&gt;
            &lt;select id="indent"&gt;
                &lt;option value="2"&gt;2 spaces&lt;/option&gt;
                &lt;option value="4" selected&gt;4 spaces&lt;/option&gt;
                &lt;option value="8"&gt;8 spaces&lt;/option&gt;
                &lt;option value="tab"&gt;Tabs&lt;/option&gt;
            &lt;/select&gt;
        &lt;/div&gt;

        &lt;div class="editor-container"&gt;
            &lt;div class="editor-panel"&gt;
                &lt;label for="inputSQL"&gt;Input SQL:&lt;/label&gt;
                &lt;textarea id="inputSQL" placeholder="Enter your SQL query here..."&gt;&lt;/textarea&gt;
            &lt;/div&gt;
            &lt;div class="editor-panel"&gt;
                &lt;label for="outputSQL"&gt;Formatted SQL:&lt;/label&gt;
                &lt;textarea id="outputSQL" readonly placeholder="Formatted SQL will appear here..."&gt;&lt;/textarea&gt;
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="controls"&gt;
            &lt;button id="formatBtn"&gt;Format SQL&lt;/button&gt;
            &lt;button id="clearBtn"&gt;Clear&lt;/button&gt;
            &lt;button id="copyBtn"&gt;Copy Formatted&lt;/button&gt;
        &lt;/div&gt;

        &lt;div class="status" id="status"&gt;Ready&lt;/div&gt;
    &lt;/div&gt;

    &lt;script&gt;
        // Enhanced SQL Formatter
        const SQLFormatter = (function() {
            'use strict';

            const TokenType = {
                WHITESPACE: 'WHITESPACE',
                WORD: 'WORD',
                STRING: 'STRING',
                RESERVED: 'RESERVED',
                RESERVED_TOPLEVEL: 'RESERVED_TOPLEVEL',
                RESERVED_NEWLINE: 'RESERVED_NEWLINE',
                OPERATOR: 'OPERATOR',
                OPEN_PAREN: 'OPEN_PAREN',
                CLOSE_PAREN: 'CLOSE_PAREN',
                LINE_COMMENT: 'LINE_COMMENT',
                BLOCK_COMMENT: 'BLOCK_COMMENT',
                NUMBER: 'NUMBER',
                PLACEHOLDER: 'PLACEHOLDER',
                DOT: 'DOT',
                COMMA: 'COMMA',
                SEMICOLON: 'SEMICOLON'
            };

            const reservedWords = [
                'ACCESS', 'ADD', 'ALL', 'ALTER', 'AND', 'ANY', 'AS', 'ASC', 'AUDIT', 'BETWEEN', 'BY', 
                'CASE', 'CHAR', 'CHECK', 'CLUSTER', 'COLUMN', 'COLUMNS', 'COMMENT', 'COMPRESS', 'CONNECT', 
                'CREATE', 'CURRENT', 'DATE', 'DECIMAL', 'DEFAULT', 'DELETE', 'DESC', 'DISTINCT', 'DROP', 
                'ELSE', 'EXCLUSIVE', 'EXISTS', 'FILE', 'FLOAT', 'FOR', 'FROM', 'GRANT', 'GROUP', 'HAVING', 
                'IDENTIFIED', 'IMMEDIATE', 'IN', 'INCREMENT', 'INDEX', 'INITIAL', 'INSERT', 'INTEGER', 
                'INTERSECT', 'INTO', 'IS', 'JOIN', 'LEFT', 'LEVEL', 'LIKE', 'LOCK', 'LONG', 'MAXEXTENTS', 
                'MINUS', 'MLSLABEL', 'MODE', 'MODIFY', 'NOAUDIT', 'NOCOMPRESS', 'NOT', 'NOTFOUND', 'NOWAIT', 
                'NULL', 'NUMBER', 'OF', 'OFFLINE', 'ON', 'ONLINE', 'OPTION', 'OR', 'ORDER', 'PCTFREE', 
                'PRIOR', 'PRIVILEGES', 'PUBLIC', 'RAW', 'RENAME', 'RESOURCE', 'REVOKE', 'RIGHT', 'ROW', 
                'ROWID', 'ROWNUM', 'ROWS', 'SELECT', 'SESSION', 'SET', 'SHARE', 'SIZE', 'SMALLINT', 'SQL', 
                'START', 'SUCCESSFUL', 'SYNONYM', 'SYSDATE', 'TABLE', 'THEN', 'TO', 'TRIGGER', 'UID', 
                'UNION', 'UNIQUE', 'UPDATE', 'USER', 'VALIDATE', 'VALUES', 'VARCHAR', 'VARCHAR2', 'VIEW', 
                'WHENEVER', 'WHERE', 'WITH'
            ];

            const reservedToplevelWords = [
                'SELECT', 'FROM', 'WHERE', 'GROUP BY', 'HAVING', 'ORDER BY', 'LIMIT', 'UNION', 'UNION ALL',
                'INSERT INTO', 'UPDATE', 'DELETE FROM', 'CREATE TABLE', 'ALTER TABLE', 'DROP TABLE',
                'VALUES', 'SET'
            ];

            const reservedNewlineWords = [
                'AND', 'OR', 'WHEN', 'ELSE', 'END', 'LEFT JOIN', 'RIGHT JOIN', 'INNER JOIN', 
                'OUTER JOIN', 'JOIN', 'ON', 'XOR', 'THEN'
            ];

            class Tokenizer {
                tokenize(input) {
                    const tokens = [];
                    let pos = 0;

                    while (pos &lt; input.length) {
                        let token = this.getNextToken(input, pos);
                        if (token) {
                            tokens.push(token);
                            pos = token.end;
                        } else {
                            pos++;
                        }
                    }
                    return tokens;
                }

                getNextToken(input, pos) {
                    const char = input[pos];

                    // Whitespace
                    if (/\s/.test(char)) {
                        return this.tokenizeWhitespace(input, pos);
                    }

                    // Comments
                    if (char === '-' &amp;&amp; input[pos + 1] === '-') {
                        return this.tokenizeLineComment(input, pos);
                    }
                    if (char === '/' &amp;&amp; input[pos + 1] === '*') {
                        return this.tokenizeBlockComment(input, pos);
                    }

                    // Strings
                    if (char === "'" || char === '"' || char === '`') {
                        return this.tokenizeString(input, pos, char);
                    }

                    // Numbers
                    if (/\d/.test(char)) {
                        return this.tokenizeNumber(input, pos);
                    }

                    // Special characters
                    if (char === '.') {
                        return { type: TokenType.DOT, value: '.', start: pos, end: pos + 1 };
                    }
                    if (char === ',') {
                        return { type: TokenType.COMMA, value: ',', start: pos, end: pos + 1 };
                    }
                    if (char === ';') {
                        return { type: TokenType.SEMICOLON, value: ';', start: pos, end: pos + 1 };
                    }
                    if (char === '(') {
                        return { type: TokenType.OPEN_PAREN, value: '(', start: pos, end: pos + 1 };
                    }
                    if (char === ')') {
                        return { type: TokenType.CLOSE_PAREN, value: ')', start: pos, end: pos + 1 };
                    }

                    // Operators
                    if (/[=&lt;&gt;!+\-*/%&amp;|~^]/.test(char)) {
                        return this.tokenizeOperator(input, pos);
                    }

                    // Words and identifiers
                    if (/[a-zA-Z_$]/.test(char)) {
                        return this.tokenizeWord(input, pos);
                    }

                    // Placeholders (like ? or :name)
                    if (char === '?' || char === ':') {
                        return this.tokenizePlaceholder(input, pos);
                    }

                    return null;
                }

                tokenizeWhitespace(input, pos) {
                    let value = '';
                    while (pos &lt; input.length &amp;&amp; /\s/.test(input[pos])) {
                        value += input[pos];
                        pos++;
                    }
                    return { type: TokenType.WHITESPACE, value, start: pos - value.length, end: pos };
                }

                tokenizeLineComment(input, pos) {
                    let value = '';
                    const start = pos;
                    while (pos &lt; input.length &amp;&amp; input[pos] !== '\n') {
                        value += input[pos];
                        pos++;
                    }
                    return { type: TokenType.LINE_COMMENT, value, start, end: pos };
                }

                tokenizeBlockComment(input, pos) {
                    let value = '';
                    const start = pos;
                    value += input[pos++]; // /
                    value += input[pos++]; // *

                    while (pos &lt; input.length &amp;&amp; !(input[pos] === '*' &amp;&amp; input[pos + 1] === '/')) {
                        value += input[pos++];
                    }

                    if (pos &lt; input.length) {
                        value += input[pos++]; // *
                        value += input[pos++]; // /
                    }

                    return { type: TokenType.BLOCK_COMMENT, value, start, end: pos };
                }

                tokenizeString(input, pos, quoteChar) {
                    let value = quoteChar;
                    const start = pos;
                    pos++;

                    while (pos &lt; input.length) {
                        if (input[pos] === quoteChar) {
                            value += quoteChar;
                            pos++;
                            break;
                        }
                        if (input[pos] === '\\') {
                            value += input[pos++];
                            if (pos &lt; input.length) {
                                value += input[pos++];
                            }
                        } else {
                            value += input[pos++];
                        }
                    }

                    return { type: TokenType.STRING, value, start, end: pos };
                }

                tokenizeNumber(input, pos) {
                    let value = '';
                    const start = pos;

                    while (pos &lt; input.length &amp;&amp; /[\d.]/.test(input[pos])) {
                        value += input[pos++];
                    }

                    return { type: TokenType.NUMBER, value, start, end: pos };
                }

                tokenizeOperator(input, pos) {
                    let value = '';
                    const start = pos;
                    const operatorChars = '=&lt;&gt;!+-*/%&amp;|~^';

                    while (pos &lt; input.length &amp;&amp; operatorChars.includes(input[pos])) {
                        value += input[pos++];
                    }

                    return { type: TokenType.OPERATOR, value, start, end: pos };
                }

                tokenizeWord(input, pos) {
                    let value = '';
                    const start = pos;

                    while (pos &lt; input.length &amp;&amp; /[\w$]/.test(input[pos])) {
                        value += input[pos++];
                    }

                    const upperValue = value.toUpperCase();
                    let type = TokenType.WORD;

                    if (reservedToplevelWords.includes(upperValue)) {
                        type = TokenType.RESERVED_TOPLEVEL;
                    } else if (reservedNewlineWords.includes(upperValue)) {
                        type = TokenType.RESERVED_NEWLINE;
                    } else if (reservedWords.includes(upperValue)) {
                        type = TokenType.RESERVED;
                    }

                    // Handle multi-word reserved words
                    if (type === TokenType.WORD) {
                        const nextTwoWords = upperValue + ' ' + (input.substring(pos, pos + 10).split(/\s+/)[0]?.toUpperCase() || '');
                        if (reservedToplevelWords.includes(nextTwoWords.trim())) {
                            type = TokenType.RESERVED_TOPLEVEL;
                            // We'll handle the multi-word matching in the formatter
                        } else if (reservedNewlineWords.includes(nextTwoWords.trim())) {
                            type = TokenType.RESERVED_NEWLINE;
                        }
                    }

                    return { type, value, start, end: pos };
                }

                tokenizePlaceholder(input, pos) {
                    let value = input[pos++];
                    const start = pos - 1;

                    if (value === ':' &amp;&amp; pos &lt; input.length &amp;&amp; /[a-zA-Z]/.test(input[pos])) {
                        while (pos &lt; input.length &amp;&amp; /[\w]/.test(input[pos])) {
                            value += input[pos++];
                        }
                    }

                    return { type: TokenType.PLACEHOLDER, value, start, end: pos };
                }
            }

            class Formatter {
                constructor(cfg = {}) {
                    this.cfg = Object.assign({
                        indent: '    ',
                        language: 'sql',
                        uppercase: true,
                        linesBetweenQueries: 1
                    }, cfg);

                    this.tokenizer = new Tokenizer();
                }

                format(query) {
                    // Split by semicolons to handle multiple statements
                    const statements = this.splitStatements(query);
                    const formattedStatements = [];

                    for (const stmt of statements) {
                        if (stmt.trim()) {
                            const formatted = this.formatSingleStatement(stmt);
                            formattedStatements.push(formatted);
                        }
                    }

                    return formattedStatements.join('\n' + ';'.repeat(this.cfg.linesBetweenQueries) + '\n');
                }

                splitStatements(query) {
                    const tokens = this.tokenizer.tokenize(query);
                    const statements = [];
                    let currentStmt = '';
                    let inString = false;
                    let stringChar = null;

                    for (let i = 0; i &lt; tokens.length; i++) {
                        const token = tokens[i];

                        if (token.type === TokenType.STRING) {
                            inString = !inString;
                            if (inString) stringChar = token.value[0];
                            currentStmt += token.value;
                        } else if (token.type === TokenType.SEMICOLON &amp;&amp; !inString) {
                            statements.push(currentStmt.trim());
                            currentStmt = '';
                        } else {
                            currentStmt += token.value;
                        }
                    }

                    if (currentStmt.trim()) {
                        statements.push(currentStmt.trim());
                    }

                    return statements;
                }

                formatSingleStatement(statement) {
                    const tokens = this.tokenizer.tokenize(statement);
                    let formatted = '';
                    let indentLevel = 0;
                    let newline = false;
                    let inSelectList = false;
                    let inFunction = false;

                    for (let i = 0; i &lt; tokens.length; i++) {
                        const token = tokens[i];
                        const prevToken = i &gt; 0 ? tokens[i - 1] : null;
                        const nextToken = i &lt; tokens.length - 1 ? tokens[i + 1] : null;

                        if (token.type === TokenType.WHITESPACE) {
                            continue;
                        }

                        // Handle dots (qualified names like table.column)
                        if (token.type === TokenType.DOT) {
                            formatted += token.value;
                            newline = false;
                            continue;
                        }

                        // Handle commas in SELECT lists
                        if (token.type === TokenType.COMMA) {
                            formatted += token.value;
                            if (inSelectList &amp;&amp; nextToken &amp;&amp; nextToken.type !== TokenType.WHITESPACE) {
                                formatted += ' ';
                            } else if (!inSelectList) {
                                newline = true;
                            }
                            continue;
                        }

                        if (token.type === TokenType.LINE_COMMENT || token.type === TokenType.BLOCK_COMMENT) {
                            if (newline) {
                                formatted += '\n' + this.cfg.indent.repeat(indentLevel);
                            }
                            formatted += token.value;
                            newline = true;
                            continue;
                        }

                        // Detect SELECT list context
                        if (token.type === TokenType.RESERVED_TOPLEVEL &amp;&amp; token.value.toUpperCase() === 'SELECT') {
                            inSelectList = true;
                        }
                        if (token.type === TokenType.RESERVED_TOPLEVEL &amp;&amp; token.value.toUpperCase() === 'FROM') {
                            inSelectList = false;
                        }

                        if (token.type === TokenType.RESERVED_TOPLEVEL) {
                            if (newline &amp;&amp; formatted.trim()) {
                                formatted += '\n';
                            }
                            formatted += '\n' + this.cfg.indent.repeat(indentLevel);
                            formatted += this.cfg.uppercase ? token.value.toUpperCase() : token.value;
                            formatted += ' ';
                            newline = false;
                            continue;
                        }

                        if (token.type === TokenType.RESERVED_NEWLINE) {
                            formatted += '\n' + this.cfg.indent.repeat(indentLevel);
                            formatted += this.cfg.uppercase ? token.value.toUpperCase() : token.value;
                            formatted += ' ';
                            newline = false;
                            continue;
                        }

                        if (token.type === TokenType.RESERVED) {
                            formatted += this.cfg.uppercase ? token.value.toUpperCase() : token.value;
                            formatted += ' ';
                            newline = false;
                            continue;
                        }

                        if (token.type === TokenType.OPEN_PAREN) {
                            // Check if this is a function call
                            const isFunction = prevToken &amp;&amp; prevToken.type === TokenType.WORD;
                            if (isFunction) {
                                inFunction = true;
                                formatted += token.value;
                            } else {
                                indentLevel++;
                                formatted += token.value;
                                newline = true;
                            }
                            continue;
                        }

                        if (token.type === TokenType.CLOSE_PAREN) {
                            if (inFunction) {
                                inFunction = false;
                                formatted += token.value;
                            } else {
                                indentLevel = Math.max(0, indentLevel - 1);
                                if (newline) {
                                    formatted += '\n' + this.cfg.indent.repeat(indentLevel);
                                }
                                formatted += token.value;
                                newline = false;
                            }
                            continue;
                        }

                        if (token.type === TokenType.OPERATOR) {
                            formatted += ' ' + token.value + ' ';
                            newline = false;
                            continue;
                        }

                        if (newline) {
                            formatted += '\n' + this.cfg.indent.repeat(indentLevel);
                            newline = false;
                        }

                        formatted += token.value;

                        // Add space after tokens except when followed by certain characters
                        if (nextToken &amp;&amp; 
                            nextToken.type !== TokenType.OPERATOR &amp;&amp; 
                            nextToken.type !== TokenType.DOT &amp;&amp;
                            nextToken.type !== TokenType.COMMA &amp;&amp;
                            nextToken.type !== TokenType.SEMICOLON &amp;&amp;
                            nextToken.type !== TokenType.CLOSE_PAREN &amp;&amp;
                            token.type !== TokenType.OPERATOR &amp;&amp;
                            token.type !== TokenType.OPEN_PAREN) {
                            formatted += ' ';
                        }
                    }

                    return formatted.trim();
                }
            }

            return {
                format: (sql, cfg) =&gt; new Formatter(cfg).format(sql),
                TokenType,
                Formatter,
                Tokenizer
            };
        })();

        // DOM handling
        document.addEventListener('DOMContentLoaded', function() {
            const inputSQL = document.getElementById('inputSQL');
            const outputSQL = document.getElementById('outputSQL');
            const formatBtn = document.getElementById('formatBtn');
            const clearBtn = document.getElementById('clearBtn');
            const copyBtn = document.getElementById('copyBtn');
            const dialectSelect = document.getElementById('dialect');
            const indentSelect = document.getElementById('indent');
            const status = document.getElementById('status');

            function formatSQL() {
                const sql = inputSQL.value.trim();
                if (!sql) {
                    outputSQL.value = '';
                    status.textContent = 'Please enter SQL to format';
                    return;
                }

                try {
                    const indent = indentSelect.value === 'tab' ? '\t' : ' '.repeat(parseInt(indentSelect.value));
                    const formatted = SQLFormatter.format(sql, {
                        indent: indent,
                        language: dialectSelect.value,
                        uppercase: true,
                        linesBetweenQueries: 2
                    });

                    outputSQL.value = formatted;
                    status.textContent = 'SQL formatted successfully';
                } catch (error) {
                    outputSQL.value = 'Error formatting SQL: ' + error.message;
                    status.textContent = 'Formatting failed';
                }
            }

            function clearAll() {
                inputSQL.value = '';
                outputSQL.value = '';
                status.textContent = 'Cleared';
            }

            function copyFormatted() {
                if (!outputSQL.value.trim()) {
                    status.textContent = 'No formatted SQL to copy';
                    return;
                }

                outputSQL.select();
                outputSQL.setSelectionRange(0, 99999);

                try {
                    navigator.clipboard.writeText(outputSQL.value);
                    status.textContent = 'Formatted SQL copied to clipboard';
                } catch (err) {
                    document.execCommand('copy');
                    status.textContent = 'Formatted SQL copied to clipboard';
                }
            }

            formatBtn.addEventListener('click', formatSQL);
            clearBtn.addEventListener('click', clearAll);
            copyBtn.addEventListener('click', copyFormatted);

            inputSQL.addEventListener('keydown', function(e) {
                if (e.ctrlKey &amp;&amp; e.key === 'Enter') {
                    formatSQL();
                }
            });

            // Better test SQL with dots and multiple statements
            inputSQL.value = `SELECT customers.name,orders.total,COUNT(*) as count FROM customers JOIN orders ON customers.id=orders.customer_id WHERE orders.total&gt;1000 GROUP BY customers.name HAVING COUNT(orders.id)&gt;5 ORDER BY orders.total DESC;UPDATE users SET last_login=NOW() WHERE id=1;SELECT table1.col1,table2.col2,func1(param1),schema2.table3.col4 FROM table1 JOIN table2 ON table1.id=table2.t1_id`;
        });
    &lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

</div>



