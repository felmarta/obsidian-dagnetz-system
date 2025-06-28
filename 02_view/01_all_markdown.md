---
tags: view
---
## Markdownファイル一覧
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
        outlinks: page?.file?.outlinks || [],
        tags: page?.file?.tags || [],
        link: page?.file?.link,
        mtime: page?.file?.mtime
    };
});

const isMarkdownFile = (file) => file.extension === 'md';

const isTargetFile = (filePath) => filePath.startsWith(ROOT_DIR);

const processMarkdownFiles = () => {
    const allFiles = app.vault.getFiles();
    const markdownFiles = new Array();
    const backlinkMap = new Map();
    const pageDataCache = new Map();
    const seen = new Set();

    for (const file of allFiles) {
        if (!isMarkdownFile(file) || !isTargetFile(file.path) || seen.has(file.path)) continue;
        seen.add(file.path);

        markdownFiles.push(file);

        const pageData = extractPageData(file.path);
        pageDataCache.set(file.path, pageData);

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

    markdownFiles.sort((a, b) => b.stat.mtime - a.stat.mtime);
    return { markdownFiles, backlinkMap, pageDataCache };
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

const getTags = (file, pageDataCache) => {
    const pageData = pageDataCache.get(file.path);
    return pageData?.tags?.length ? pageData.tags.join('\n') : '';
};

const createTableRow = (file, backlinkMap, pageDataCache) => {
    const pageData = pageDataCache.get(file.path);

    return [
        pageData?.link || `[[${file.path}|${file.name}]]`,
        getTags(file, pageDataCache),
        getBacklinks(file.path, backlinkMap),
        pageData?.mtime || new Date(file.stat.mtime)
    ];
};

const createTableData = (files, backlinkMap, pageDataCache) =>
    files.map(file => createTableRow(file, backlinkMap, pageDataCache));

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
        dv.table(["ファイル", "タグ", "被リンク", "更新日時"], batch);
    }
};

const executeMarkdownList = () => {
    try {
        const { markdownFiles, backlinkMap, pageDataCache } = processMarkdownFiles();
        const tableData = createTableData(markdownFiles, backlinkMap, pageDataCache);
        renderSection(dv, markdownFiles, tableData);
    } catch (error) {
        dv.paragraph(`エラーが発生しました: ${error.message}`);
    } finally {
        if (stringPool.size > 0) {
            stringPool.clear();
        }
    }
};

executeMarkdownList();
```
