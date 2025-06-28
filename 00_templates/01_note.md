---
created: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
tags:
---

<%*
try {
  // 設定と日付パス作成
  const basePath = "01_data";
  const datePath = tp.date.now("YYYY/MM/DD");
  const folderPath = `${basePath}/${datePath}`;
  // フォルダを直接作成
  await app.vault.createFolder(folderPath).catch(() => {});
  // ユニークなファイル名を生成する関数
  async function getUniqueFileName(folderPath, baseName) {
    let fileName = baseName;
    let counter = 1;
    // ファイルが存在する限り番号を増やしていく
    while (app.vault.getAbstractFileByPath(`${folderPath}/${fileName}.md`)) {
      fileName = `${baseName} ${counter}`;
      counter++;
    }
    return fileName;
  }
  // 元のファイル名
  const originalName = tp.file.title;
  // ユニークなファイル名を取得
  const uniqueName = await getUniqueFileName(folderPath, originalName);
  // ファイルを移動
  await tp.file.move(`${folderPath}/${uniqueName}`);
  // 名前が変更された場合は通知
  if (originalName !== uniqueName) {
    new Notice(`ファイル名が重複していたため "${uniqueName}" に変更されました`);
  }
} catch (error) {
  new Notice(`エラー: ${error.message}`);
}
%>
