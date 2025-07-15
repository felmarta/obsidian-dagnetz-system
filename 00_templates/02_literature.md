---
created: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
tags:
  - 文献
title: 
source: 
author: 
published: 
description:
---
<%*
const CONFIG = {
  BASE_FOLDER: "01_data",
  DATE_FORMAT: "YYYY/MM/DD"
}

const createFolderIfNotExists = async (path) => {
  try {
    await app.vault.createFolder(path)
  } catch {
    // フォルダが既に存在
  }
}

const getUniqueName = async (folder, base) => {
  let name = base
  let i = 1
  while (app.vault.getAbstractFileByPath(`${folder}/${name}.md`)) {
    name = `${base} ${i++}`
  }
  return name
}

const moveFileToFolder = async (fileTitle, folder) => {
  await createFolderIfNotExists(folder)
  const uniqueName = await getUniqueName(folder, fileTitle)
  await tp.file.move(`${folder}/${uniqueName}`)
  return uniqueName
}

const notifyIfRenamed = (original, newName) => {
  if (original !== newName) {
    new Notice(`ファイル名が重複していたため "${newName}" に変更されました`)
  }
}

try {
  const folderPath = `${CONFIG.BASE_FOLDER}/${tp.date.now(CONFIG.DATE_FORMAT)}`
  const originalName = tp.file.title
  const newName = await moveFileToFolder(originalName, folderPath)
  notifyIfRenamed(originalName, newName)
} catch (error) {
  new Notice(`エラー: ${error.message}`)
}
%>