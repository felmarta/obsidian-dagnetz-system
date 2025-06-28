---
tags: view
---
## 添付ファイル一覧
```dataviewjs
const ROOT_DIR = "01_data";
const DISPLAY_SIZE = 50;

const memoizeOneArg = (fn) => {
    const cache = new Map();
    return (arg) => {
        if (cache.has(arg)) return cache.get(arg);
        const result = fn(arg);
        cache.set(arg, result);
        return result;
    };
};

const formatDateTime = (date) => {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    const hours = String(date.getHours()).padStart(2, '0');
    const minutes = String(date.getMinutes()).padStart(2, '0');
    const seconds = String(date.getSeconds()).padStart(2, '0');
    return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
};

const createFileLink = (file) => `[[${file.path}|${file.name}]]`;

const getExtension = (file) => file.extension || '';

const extractFileName = (path) => {
    const lastSlash = path.lastIndexOf('/');
    const fileName = lastSlash === -1 ? path : path.slice(lastSlash + 1);
    return fileName.endsWith('.md') ? fileName.slice(0, -3) : fileName;
};

const stringPool = new Map();
const pooledString = (str) => {
    if (stringPool.has(str)) return stringPool.get(str);
    stringPool.set(str, str);
    return str;
};

const createWikiLink = (path, name) => {
    return pooledString(`[[${path}|${extractFileName(name)}]]`);
};

const extractPageData = memoizeOneArg((filePath) => {
    const page = dv.page(filePath);
    return {
        page,
        outlinks: page?.file?.outlinks || []
    };
});

const isMarkdownFile = (file) => file.extension === 'md';
const isAttachmentFile = (file) => file.extension !== 'md';
const isTargetFile = (filePath) => filePath.startsWith(ROOT_DIR);

const processAttachmentFiles = () => {
    const allFiles = app.vault.getFiles();
    const attachmentFiles = new Array();
    const backlinkMap = new Map();
    const seen = new Set();

    for (const file of allFiles) {
        if (!isTargetFile(file.path) || seen.has(file.path)) continue;
        seen.add(file.path);

        if (isAttachmentFile(file)) {
            attachmentFiles.push(file);
        }

        if (isMarkdownFile(file)) {
            const pageData = extractPageData(file.path);

            pageData.outlinks.forEach(outlink => {
                if (!outlink?.path) return;

                if (!backlinkMap.has(outlink.path)) {
                    backlinkMap.set(outlink.path, []);
                }
                backlinkMap.get(outlink.path).push({
                    path: file.path,
                    name: file.name
                });
            });
        }
    }
    attachmentFiles.sort((a, b) => b.stat.mtime - a.stat.mtime);
    return { attachmentFiles, backlinkMap };
};

const getBacklinks = (filePath, backlinkMap) => {
    const backlinks = backlinkMap.get(filePath);
    if (!backlinks?.length) return "";

    const seen = new Set();
    const result = [];

    for (const link of backlinks) {
        if (!link?.path) continue;

        const wikiLink = createWikiLink(link.path, link.name || link.path);
        if (!seen.has(wikiLink)) {
            seen.add(wikiLink);
            result.push(wikiLink);
        }
    }

    return result.join('\n');
};

const getDirectoryPath = (filePath) => {
    const lastSlash = filePath.lastIndexOf('/');
    return lastSlash === -1 ? '' : filePath.slice(0, lastSlash);
};

const createAttachmentTableRow = (file, backlinkMap) => [
    createFileLink(file),
    getExtension(file),
    getBacklinks(file.path, backlinkMap),
    getDirectoryPath(file.path),
    formatDateTime(new Date(file.stat.mtime))
];

const createAttachmentTableData = (files, backlinkMap) =>
    files.map(file => createAttachmentTableRow(file, backlinkMap));

const renderSection = (dv, files, tableData) => {
    if (!files?.length) {
        dv.paragraph("該当なし");
        return;
    }

    const totalCount = files.length;
    dv.paragraph(`**${totalCount}件** - 更新日時 降順`);

    for (let i = 0; i < totalCount; i += DISPLAY_SIZE) {
        const batch = tableData.slice(i, i + DISPLAY_SIZE);
        const batchStart = i + 1;
        const batchEnd = Math.min(i + DISPLAY_SIZE, totalCount);

        dv.paragraph(`**${batchStart}-${batchEnd}件目**`);
        dv.table([
            "ファイル", "拡張子", "被リンク", "ディレクトリ", "更新日時"
        ], batch);
    }
};

const executeAttachmentList = () => {
    try {
        const { attachmentFiles, backlinkMap } = processAttachmentFiles();
        const tableData = createAttachmentTableData(attachmentFiles, backlinkMap);
        renderSection(dv, attachmentFiles, tableData);
    } catch (error) {
        dv.paragraph(`エラーが発生しました: ${error.message}`);
    } finally {
        if (stringPool.size > 0) {
            stringPool.clear();
        }
    }
};

executeAttachmentList();
```
