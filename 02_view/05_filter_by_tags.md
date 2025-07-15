---
tags: view
---
```dataviewjs
const ROOT_DIR = "01_data";
const DISPLAY_SIZE = 50;

// 複数タグ対応（空の配列の場合は全てのファイルを表示）
// 例1: ["work"] ; タグ1つ
// 例2: ["work", "personal"] ; 複数タグ
// 例3: [] ; タグ指定なし
const FILTER_TAGS = ["sample"];

// 除外したいタグを指定（空の配列の場合は除外なし）
// 例1: ["完了"] ; タグ1つ除外
// 例2: ["完了", "アーカイブ"] ; 複数タグ除外
// 例3: [] ; 除外なし
const EXCLUDE_TAGS = ["complete"];

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

const processBacklinks = (inlinks) => {
    if (!inlinks?.length) return "";

    const seen = new Set();
    const result = inlinks.array()
        .filter(link => link?.path)
        .map(link => createWikiLink(link.path, link.path))
        .filter(wikiLink => {
            if (seen.has(wikiLink)) return false;
            seen.add(wikiLink);
            return true;
        });

    return result.length ? result.join('\n') : "";
};

const processOutlinks = (outlinks) => {
    if (!outlinks?.length) return "";

    const seen = new Set();
    const result = outlinks.array()
        .filter(link => link?.path)
        .map(link => createWikiLink(link.path, link.path))
        .filter(wikiLink => {
            if (seen.has(wikiLink)) return false;
            seen.add(wikiLink);
            return true;
        });

    return result.length ? result.join('\n') : "";
};

const createTableRow = (page) => {
    const link = page.file.link;
    const outlinks = processOutlinks(page.file.outlinks);
    const backlinks = processBacklinks(page.file.inlinks);
    const mtime = page.file.mtime;

    return [link, outlinks, backlinks, mtime];
};

const buildQuery = () => {
    let query = `"${ROOT_DIR}"`;

    if (FILTER_TAGS && FILTER_TAGS.length > 0) {
        const tagQuery = FILTER_TAGS.map(tag => `#${tag}`).join(' or ');
        query += ` and (${tagQuery})`;
    }
    
    if (EXCLUDE_TAGS && EXCLUDE_TAGS.length > 0) {
        const excludeQuery = EXCLUDE_TAGS.map(tag => `-#${tag}`).join(' and ');
        query += ` and ${excludeQuery}`;
    }
    
    return query;
};

const renderSection = (pages) => {
    const totalCount = pages.length;
    let filterText = "";

    const filterParts = [];
    
    if (FILTER_TAGS && FILTER_TAGS.length > 0) {
        filterParts.push(`含む: #${FILTER_TAGS.join(' #')}`);
    }
    
    if (EXCLUDE_TAGS && EXCLUDE_TAGS.length > 0) {
        filterParts.push(`除外: #${EXCLUDE_TAGS.join(' #')}`);
    }
    
    if (filterParts.length > 0) {
        filterText = ` (${filterParts.join(' | ')})`;
    }

    dv.header(2, `ノート一覧${filterText}`);
    dv.paragraph(`**${totalCount}**件 - 更新日時 降順`);

    if (!pages?.length) {
        dv.paragraph("該当なし");
        return;
    }
    
    for (let i = 0; i < totalCount; i += DISPLAY_SIZE) {
        const batchEnd = Math.min(i + DISPLAY_SIZE, totalCount);
        const batch = pages.slice(i, batchEnd).map(createTableRow);
        const batchStart = i + 1;

        dv.paragraph(`**${batchStart}-${batchEnd}件目**`);
        dv.table(["ファイル", "リンク先", "被リンク", "更新日時"], batch);
    }
};

const executeMarkdownList = () => {
    try {
        const query = buildQuery();
        const pages = dv.pages(query)
            .sort(p => p.file.mtime, 'desc')
            .array();
        
        renderSection(pages);
    } catch (error) {
        dv.paragraph(`エラーが発生しました: ${error.message}`);
        console.error('DataviewJS エラー:', error);
    } finally {
        stringPool.clear();
        fileNameCache.clear();
    }
};

executeMarkdownList();
```
