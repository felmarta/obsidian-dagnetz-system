---
tags: view
---
## 添付ファイル一覧
```dataviewjs
const ROOT_DIR = "01_data";
const DISPLAY_SIZE = 50;

const stringPool = new Map();
const fileNameCache = new Map();

const pooledString = (str) => {
    if (stringPool.has(str)) return stringPool.get(str);
    stringPool.set(str, str);
    return str;
};

const extractFileName = (path) => {
    if (fileNameCache.has(path)) return fileNameCache.get(path);

    const lastSlash = path.lastIndexOf('/');
    const fileName = lastSlash === -1 ? path : path.slice(lastSlash + 1);
    const result = fileName.endsWith('.md') ? fileName.slice(0, -3) : fileName;

    fileNameCache.set(path, result);
    return result;
};

const createWikiLink = (path, name) => {
    return pooledString(`[[${path}|${extractFileName(name)}]]`);
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
const isAttachmentFile = (file) => file.extension !== 'md';
const isTargetFile = (filePath) => filePath.startsWith(ROOT_DIR);

const buildBacklinkMap = () => {
    const backlinkMap = new Map();

    const pages = dv.pages(`"${ROOT_DIR}"`)
        .array();

    pages.forEach(page => {
        const outlinks = page.file.outlinks?.array() || [];
        
        outlinks
            .filter(outlink => outlink?.path)
            .forEach(outlink => {
                if (!backlinkMap.has(outlink.path)) {
                    backlinkMap.set(outlink.path, []);
                }
                backlinkMap.get(outlink.path).push({
                    path: page.file.path,
                    name: page.file.name
                });
            });
    });

    return backlinkMap;
};

const getAttachmentFiles = () => {
    const allFiles = app.vault.getFiles();
    const attachmentFiles = [];
    const seen = new Set();

    for (const file of allFiles) {
        if (!isTargetFile(file.path) || seen.has(file.path)) continue;
        seen.add(file.path);

        if (isAttachmentFile(file)) {
            attachmentFiles.push(file);
        }
    }

    attachmentFiles.sort((a, b) => b.stat.mtime - a.stat.mtime);
    return attachmentFiles;
};

const getBacklinks = (filePath, backlinkMap) => {
    const backlinks = backlinkMap.get(filePath);
    if (!backlinks?.length) return "";

    const seen = new Set();
    const result = backlinks
        .filter(link => link?.path)
        .map(link => createWikiLink(link.path, link.name || link.path))
        .filter(wikiLink => {
            if (seen.has(wikiLink)) return false;
            seen.add(wikiLink);
            return true;
        });

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

const renderSection = (files, tableData) => {
    const totalCount = files.length;
    dv.paragraph(`**${totalCount}件** - 更新日時 降順`);

    if (!files?.length) {
        dv.paragraph("該当なし");
        return;
    }

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
        const backlinkMap = buildBacklinkMap();
        const attachmentFiles = getAttachmentFiles();
        const tableData = attachmentFiles.map(file => 
            createAttachmentTableRow(file, backlinkMap)
        );
        
        renderSection(attachmentFiles, tableData);
    } catch (error) {
        dv.paragraph(`エラーが発生しました: ${error.message}`);
    } finally {
        stringPool.clear();
        fileNameCache.clear();
    }
};

executeAttachmentList();
```
