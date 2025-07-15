---
tags: view
---
## Markdownファイル一覧
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

const processTags = (pageData) => {
    const tags = pageData?.file?.tags;
    return tags?.length ? tags.join('\n') : '';
};

const createTableRow = (page) => {
    const link = page.file.link;
    const tags = processTags(page);
    const backlinks = processBacklinks(page.file.inlinks);
    const mtime = page.file.mtime;

    return [link, tags, backlinks, mtime];
};

const renderSection = (pages) => {
    if (!pages?.length) {
        dv.paragraph("該当なし");
        return;
    }

    const totalCount = pages.length;
    dv.paragraph(`**${totalCount}件** - 更新日時 降順`);

    for (let i = 0; i < totalCount; i += DISPLAY_SIZE) {
        const batchEnd = Math.min(i + DISPLAY_SIZE, totalCount);
        const batch = pages.slice(i, batchEnd).map(createTableRow);
        const batchStart = i + 1;

        dv.paragraph(`**${batchStart}-${batchEnd}件目**`);
        dv.table(["ファイル", "タグ", "被リンク", "更新日時"], batch);
    }
};

const executeMarkdownList = () => {
    try {
        const pages = dv.pages(`"${ROOT_DIR}"`)
            .sort(p => p.file.mtime, 'desc')
            .array();
        
        renderSection(pages);
    } catch (error) {
        dv.paragraph(`エラーが発生しました: ${error.message}`);
    } finally {
        stringPool.clear();
        fileNameCache.clear();
    }
};

executeMarkdownList();

```
