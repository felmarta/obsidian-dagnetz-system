---
tags: view
---
```dataviewjs
const ROOT_DIR = '01_data';
const DAYS = 90;  // 日数
const DISPLAY_SIZE = 50;  // 表示件数

const memoizeZeroArg = (fn) => {
    let cached = false;
    let result;
    return () => {
        if (!cached) {
            result = fn();
            cached = true;
        }
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

const generateDatePaths = memoizeZeroArg(() => {
    const datePaths = new Set();
    const currentDate = new Date();

    for (let i = 0; i < DAYS; i++) {
        const date = new Date(currentDate);
        date.setDate(date.getDate() - i);
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0');
        const day = String(date.getDate()).padStart(2, '0');
        const datePath = `${ROOT_DIR}/${year}/${month}/${day}`;
        datePaths.add(datePath);
    }
    return datePaths;
});

const getCurrentDateString = memoizeZeroArg(() => {
    const today = new Date();
    const year = today.getFullYear();
    const month = String(today.getMonth() + 1).padStart(2, '0');
    const day = String(today.getDate()).padStart(2, '0');
    return `${year}/${month}/${day}`;
});

const createPathMatcher = memoizeZeroArg(() => {
    const escapedRootDir = ROOT_DIR.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    return new RegExp(`^${escapedRootDir}/(\\d{4})/(\\d{2})/(\\d{2})`);
});

const isFileRelevant = (filePath, datePaths) => {
    const pattern = createPathMatcher();
    const pathMatch = filePath.match(pattern);

    if (!pathMatch) return false;

    const fileDate = `${ROOT_DIR}/${pathMatch[1]}/${pathMatch[2]}/${pathMatch[3]}`;
    return datePaths.has(fileDate);
};

const processFiles = () => {
    const datePaths = generateDatePaths();
    const todayStr = getCurrentDateString();
    const todayPath = `${ROOT_DIR}/${todayStr}`;

    const todayFiles = [];
    const recentFiles = [];
    const backlinkMap = new Map();
    const outlinkMap = new Map();
    const pageDataCache = new Map();

    const allPages = dv.pages(`"${ROOT_DIR}"`)
        .array()
        .filter(page => isFileRelevant(page.file.path, datePaths));

    allPages.forEach(page => {
        const file = app.vault.getAbstractFileByPath(page.file.path);
        const isToday = page.file.path.startsWith(todayPath);
        
        (isToday ? todayFiles : recentFiles).push(file);

        pageDataCache.set(page.file.path, {
            page,
            outlinks: page.file.outlinks || [],
            tags: page.file.tags || []
        });
        
        outlinkMap.set(page.file.path, page.file.outlinks || []);

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

    const allFiles = app.vault.getFiles();
    const seen = new Set(allPages.map(page => page.file.path));
    
    allFiles
        .filter(file => file.extension !== 'md' && 
                      isFileRelevant(file.path, datePaths) && 
                      !seen.has(file.path))
        .forEach(file => {
            const isToday = file.path.startsWith(todayPath);
            (isToday ? todayFiles : recentFiles).push(file);
        });

    const sortByMtime = (a, b) => b.stat.mtime - a.stat.mtime;
    return {
        todayFiles: todayFiles.sort(sortByMtime),
        recentFiles: recentFiles.sort(sortByMtime),
        backlinkMap,
        outlinkMap,
        pageDataCache
    };
};

const formatLinks = (links) => {
    if (!links?.length) return "";

    const seen = new Set();
    const result = links
        .filter(link => link?.path)
        .map(link => createWikiLink(link.path, link.name || link.path))
        .filter(wikiLink => {
            if (seen.has(wikiLink)) return false;
            seen.add(wikiLink);
            return true;
        });

    return result.join('\n');
};

const getBacklinks = (filePath, backlinkMap) =>
    formatLinks(backlinkMap.get(filePath) || []);

const getOutlinks = (filePath, outlinkMap) => {
    const outlinks = outlinkMap.get(filePath) || [];
    if (!outlinks.length) return "";

    const seen = new Set();
    const result = outlinks.array()
        .filter(link => link?.path)
        .map(link => createWikiLink(link.path, link.path))
        .filter(wikiLink => {
            if (seen.has(wikiLink)) return false;
            seen.add(wikiLink);
            return true;
        });

    return result.join('\n');
};

const getTags = (file, pageDataCache) => {
    if (file.extension !== 'md') return '';

    const pageData = pageDataCache.get(file.path);
    if (!pageData?.tags?.length) return '';

    return pageData.tags.join('\n');
};

const createTableRow = (file, backlinkMap, outlinkMap, pageDataCache) => [
    createFileLink(file),
    getTags(file, pageDataCache),
    getOutlinks(file.path, outlinkMap),
    getBacklinks(file.path, backlinkMap),
    formatDateTime(new Date(file.stat.mtime))
];

const createTableData = (files, backlinkMap, outlinkMap, pageDataCache) =>
    files.map(file => createTableRow(file, backlinkMap, outlinkMap, pageDataCache));

const renderSection = (title, files, tableData) => {
    dv.header(2, title);

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
            "ファイル名", "タグ", "リンク先", "被リンク", "更新日時"
        ], batch);
    }
};

const formatDateRange = (startDate, endDate, days) =>
    `${startDate.toLocaleDateString()} ～ ${endDate.toLocaleDateString()} （${days}日間）`;

const executeView = () => {
    try {
        // Step 1: ファイル情報取得
        const {
            todayFiles,
            recentFiles,
            backlinkMap,
            outlinkMap,
            pageDataCache
        } = processFiles();

        // Step 2: テーブルデータ生成
        const todayTableData = createTableData(todayFiles, backlinkMap, outlinkMap, pageDataCache);
        const recentTableData = createTableData(recentFiles, backlinkMap, outlinkMap, pageDataCache);

        // Step 3: 表示 今日
        const today = new Date();
        renderSection(today.toLocaleDateString(), todayFiles, todayTableData);

        // Step 4: 表示 過去分
        const currentDate = new Date();
        currentDate.setDate(currentDate.getDate() - 1);
        const startDate = new Date(currentDate);
        const endDate = new Date(currentDate);
        startDate.setDate(startDate.getDate() - (DAYS - 1));
        const recentTitle = formatDateRange(startDate, endDate, DAYS);
        renderSection(recentTitle, recentFiles, recentTableData);
    } catch (error) {
        dv.paragraph(`エラーが発生しました: ${error.message}`);
    } finally {
        stringPool.clear();
    }
};

executeView();
```
