---
sidebar: false
---

# pnpm patch 修改第三方包源码

element-plus 2.4.4 版本出现之前没有的bug：el-date-picker type="datetime"时，切换月份后选日期会选中1日，再点一下才能选中点击的日期 #247。看到 issues 里升级版本可以解决，但是升级版本又带来其他问题，看了升级的源码，只改了两行代码，于是通过打补丁的方式来解决。

了解了 patch-package 这个工具。刚尝试了一下，在pnpm管理下，无法正常使用，原来pnpm官方新增了两个命令来处理这个问题：pnpm patch xxx@xxx (--edit-dir xxx)和pnpm patch-commit dir，实现原理与patch-package应该是一致的。

```text
patch-package 6.4.7

**ERROR** No package-lock.json, npm-shrinkwrap.json, or yarn.lock file.

You must use either npm@>=5, yarn, or npm-shrinkwrap to manage this project's
dependencies.
```

## 使用

先使用 `pnpm patch xxx@xxx`命令，生成修改该包的一个临时路径，我们可以使用--edit-dir xxx来自定义路径。

```shell
pnpm patch element-plus@2.4.4
```

```shell
You can now edit the following folder: C:\Users\Lenovo\AppData\Local\Temp\1fd0b80cd1fa4fd7914a51492943373d
```

随后，我们可以再该目录下，对包进行修改：

```js
 const handleDatePick = async (value, keepOpen) => {
  if (selectionMode.value === "date") {
    value = value;
    let newDate = props.parsedValue ? props.parsedValue.year(value.year()).month(value.month()).date(value.date()) : value;
    if (!checkDateWithinRange(newDate)) {
      newDate = selectableRange.value[0][0].year(value.year()).month(value.month()).date(value.date());
    }
    innerDate.value = newDate;
    emit(newDate, showTime.value || keepOpen);
    if (props.type === "datetime") {
      // 新增
      await nextTick()
      handleFocusPicker();
    }
  } else if (selectionMode.value === "week") {
    emit(value.date);
  } else if (selectionMode.value === "dates") {
    emit(value, true);
  }
};
```

接着，我们运行pnpm patch-commit xxx，xxx指的是刚才的临时路径。

```shell
pnpm patch-commit 'C:\Users\Lenovo\AppData\Local\Temp\1fd0b80cd1fa4fd7914a51492943373d'
```

可以在项目目录下看到，生成了一个patches文件夹，里面包含了一个修改记录，同时在package.json中多了一个配置项。

这样子，我们的修改就完成了，在终端运行也可以看到我们修改后的结果。

## 源码

patch：

```ts
// 每一个命令都通过handler函数来执行
export async function handler (opts: PatchCommandOptions, params: string[]) {
  // 判断用户是否传了路径，并且是空文件路径
  if (opts.editDir && fs.existsSync(opts.editDir) && fs.readdirSync(opts.editDir).length > 0) {
    throw new PnpmError('PATCH_EDIT_DIR_EXISTS', `The target directory already exists: '${opts.editDir}'`)
  }
  // 使用用户路径，或者使用一个临时路径
  const editDir = opts.editDir ?? tempy.directory()
  // 往该路径写入要修改的包内容
  await writePackage(params[0], editDir, opts)
  // 返回提示
  return `You can now edit the following folder: ${editDir}`
}
```

patch-commit：

```ts
export async function handler (opts: install.InstallCommandOptions, params: string[]) {
  // 获得目标路径
  const userDir = params[0]
  // 获取patches路径，并创建该文件夹
  const lockfileDir = opts.lockfileDir ?? opts.dir ?? process.cwd()
  const patchesDir = path.join(lockfileDir, 'patches')
  await fs.promises.mkdir(patchesDir, { recursive: true })
  // 获取要求改包的package.json，用于获取包的name和version
  const patchedPkgManifest = await readPackageJsonFromDir(userDir)
  const pkgNameAndVersion = `${patchedPkgManifest.name}@${patchedPkgManifest.version}`
  // 创建临时目录，并写出原始包的内容
  const srcDir = tempy.directory()
  await writePackage(pkgNameAndVersion, srcDir, opts)
  // 对比两个目录下的内容，获得对比后的结果
  const patchContent = await diffFolders(srcDir, userDir)
  // 生成并写入patch文件
  const patchFileName = pkgNameAndVersion.replace('/', '__')
  await fs.promises.writeFile(path.join(patchesDir, `${patchFileName}.patch`), patchContent, 'utf8')
  // 修改项目packag.json中的内容
  let { manifest, writeProjectManifest } = await tryReadProjectManifest(lockfileDir)
  if (!manifest) {
    manifest = {}
  }
  if (!manifest.pnpm) {
    manifest.pnpm = {
      patchedDependencies: {},
    }
  } else if (!manifest.pnpm.patchedDependencies) {
    manifest.pnpm.patchedDependencies = {}
  }
  manifest.pnpm.patchedDependencies![pkgNameAndVersion] = `patches/${patchFileName}.patch`
  await writeProjectManifest(manifest)
  // 执行`pnpm install`来应用更改
  return install.handler(opts)
}
```

如何对比两个目录下的内容？

```ts
async function diffFolders (folderA: string, folderB: string) {
  // 对路径进行格式化处理
  const folderAN = folderA.replace(/\\/g, '/')
  const folderBN = folderB.replace(/\\/g, '/')
  let stdout!: string
  let stderr!: string

  try {
	// 通过git命令来对目录进行对比
	// -c core.safecrlf=false 表示允许提交包含混合换行符的文件，主要因为window和linux等系统的换行符不同
	// diff [<options>] --no-index <path1> <path2> 对比文件系统上两个目录下的差异
	// --src-prefix=a/ 源目录前缀
	// --dst-prefix=b/ 修改后目录前缀
	// --ignore-cr-at-eol 比较时忽略行尾的回车
	// 更多详见：https://git-scm.com/docs/diff-options
    const result = await execa('git', ['-c', 'core.safecrlf=false', 'diff', '--src-prefix=a/', '--dst-prefix=b/', '--ignore-cr-at-eol', '--irreversible-delete', '--full-index', '--no-index', '--text', folderAN, folderBN], {
      cwd: process.cwd(),
      env: {
        ...process.env,
        // #region Predictable output
        // These variables aim to ignore the global git config so we get predictable output
        // https://git-scm.com/docs/git#Documentation/git.txt-codeGITCONFIGNOSYSTEMcode
        GIT_CONFIG_NOSYSTEM: '1',
        HOME: '',
        XDG_CONFIG_HOME: '',
        USERPROFILE: '',
        // #endregion
      },
    })
    stdout = result.stdout
    stderr = result.stderr
  } catch (err: any) { // eslint-disable-line
    stdout = err.stdout
    stderr = err.stderr
  }
  // we cannot rely on exit code, because --no-index implies --exit-code
  // i.e. git diff will exit with 1 if there were differences
  if (stderr.length > 0)
    throw new Error(`Unable to diff directories. Make sure you have a recent version of 'git' available in PATH.\nThe following error was reported by 'git':\n${stderr}`)

  return stdout
    .replace(new RegExp(`(a|b)(${escapeStringRegexp(`/${removeTrailingAndLeadingSlash(folderAN)}/`)})`, 'g'), '$1/')
    .replace(new RegExp(`(a|b)${escapeStringRegexp(`/${removeTrailingAndLeadingSlash(folderBN)}/`)}`, 'g'), '$1/')
    .replace(new RegExp(escapeStringRegexp(`${folderAN}/`), 'g'), '')
    .replace(new RegExp(escapeStringRegexp(`${folderBN}/`), 'g'), '')
    .replace(/\n\\ No newline at end of file$/, '')
}

function removeTrailingAndLeadingSlash (p: string) {
  if (p.startsWith('/') || p.endsWith('/')) {
    return p.replace(/^\/|\/$/g, '')
  }
  return p
}
```

我们看看生成后的内容是什么：

```patch
diff --git a/es/components/date-picker/src/date-picker-com/panel-date-pick.mjs b/es/components/date-picker/src/date-picker-com/panel-date-pick.mjs
index e957be691ee71231d93a8937c821fa8e114ba3d6..3c8100c13f41dd34ab7de21c8a37c8666d904d81 100644
--- a/es/components/date-picker/src/date-picker-com/panel-date-pick.mjs
+++ b/es/components/date-picker/src/date-picker-com/panel-date-pick.mjs
@@ -86,7 +86,7 @@ const _sfc_main = /* @__PURE__ */ defineComponent({
       isChangeToNow.value = false;
       isShortcut = false;
     };
-    const handleDatePick = (value, keepOpen) => {
+    const handleDatePick = async (value, keepOpen) => {
       if (selectionMode.value === "date") {
         value = value;
         let newDate = props.parsedValue ? props.parsedValue.year(value.year()).month(value.month()).date(value.date()) : value;
@@ -96,6 +96,7 @@ const _sfc_main = /* @__PURE__ */ defineComponent({
         innerDate.value = newDate;
         emit(newDate, showTime.value || keepOpen);
         if (props.type === "datetime") {
+          await nextTick()
           handleFocusPicker();
         }
       } else if (selectionMode.value === "week") {
```

那么，当我们运行pnpm install时，会去检测package.json中的内容，应用补丁。而在pnpm中，也是使用了patch-package中的api进行处理。

```ts
import { applyPatch } from 'patch-package/dist/applyPatches'

function applyPatchToDep (patchDir: string, patchFilePath: string) {
  // Ideally, we would just run "patch" or "git apply".
  // However, "patch" is not available on Windows and "git apply" is hard to execute on a subdirectory of an existing repository
  const cwd = process.cwd()
  process.chdir(patchDir)
  const success = applyPatch({
    patchFilePath,
    patchDir,
  })
  process.chdir(cwd)
  if (!success) {
    throw new PnpmError('PATCH_FAILED', `Could not apply patch ${patchFilePath} to ${patchDir}`)
  }
}
```

开始处理补丁。

```ts
export function applyPatch({
  patchFilePath,
  reverse,
  packageDetails,
  patchDir,
}: {
  patchFilePath: string
  reverse: boolean
  packageDetails: PackageDetails
  patchDir: string
}): boolean {
  // 读取，解析补丁
  const patch = readPatch({ patchFilePath, packageDetails, patchDir })
  // 处理所有补丁解析后的effect对象
  try {
    executeEffects(reverse ? reversePatch(patch) : patch, { dryRun: false })
  } catch (e) {
    try {
      executeEffects(reverse ? patch : reversePatch(patch), { dryRun: true })
    } catch (e) {
      return false
    }
  }

  return true
}
```

对每一行进行解析，最后得到一个包含diff信息的对象

```ts
function parsePatchLines(
  lines: string[],
  { supportLegacyDiffs }: { supportLegacyDiffs: boolean },
): FileDeets[] {
  const result: FileDeets[] = []
  let currentFilePatch: FileDeets = emptyFilePatch()
  let state: State = "parsing header"
  let currentHunk: Hunk | null = null
  let currentHunkMutationPart: PatchMutationPart | null = null

  function commitHunk() {
    if (currentHunk) {
      if (currentHunkMutationPart) {
        currentHunk.parts.push(currentHunkMutationPart)
        currentHunkMutationPart = null
      }
      currentFilePatch.hunks!.push(currentHunk)
      currentHunk = null
    }
  }

  function commitFilePatch() {
    commitHunk()
    result.push(currentFilePatch)
    currentFilePatch = emptyFilePatch()
  }
  // 对每一行进行解析
  for (let i = 0; i < lines.length; i++) {
    const line = lines[i]

    if (state === "parsing header") {
      if (line.startsWith("@@")) {
        state = "parsing hunks"
        currentFilePatch.hunks = []
        i--
      } else if (line.startsWith("diff --git ")) {
        if (currentFilePatch && currentFilePatch.diffLineFromPath) {
          commitFilePatch()
        }
        const match = line.match(/^diff --git a\/(.*?) b\/(.*?)\s*$/)
        if (!match) {
          throw new Error("Bad diff line: " + line)
        }
        currentFilePatch.diffLineFromPath = match[1]
        currentFilePatch.diffLineToPath = match[2]
      } else if (line.startsWith("old mode ")) {
        currentFilePatch.oldMode = line.slice("old mode ".length).trim()
      } else if (line.startsWith("new mode ")) {
        currentFilePatch.newMode = line.slice("new mode ".length).trim()
      } else if (line.startsWith("deleted file mode ")) {
        currentFilePatch.deletedFileMode = line
          .slice("deleted file mode ".length)
          .trim()
      } else if (line.startsWith("new file mode ")) {
        currentFilePatch.newFileMode = line
          .slice("new file mode ".length)
          .trim()
      } else if (line.startsWith("rename from ")) {
        currentFilePatch.renameFrom = line.slice("rename from ".length).trim()
      } else if (line.startsWith("rename to ")) {
        currentFilePatch.renameTo = line.slice("rename to ".length).trim()
      } else if (line.startsWith("index ")) {
        const match = line.match(/(\w+)\.\.(\w+)/)
        if (!match) {
          continue
        }
        currentFilePatch.beforeHash = match[1]
        currentFilePatch.afterHash = match[2]
      } else if (line.startsWith("--- ")) {
        currentFilePatch.fromPath = line.slice("--- a/".length).trim()
      } else if (line.startsWith("+++ ")) {
        currentFilePatch.toPath = line.slice("+++ b/".length).trim()
      }
    } else {
      if (supportLegacyDiffs && line.startsWith("--- a/")) {
        state = "parsing header"
        commitFilePatch()
        i--
        continue
      }
      // parsing hunks
      const lineType = hunkLinetypes[line[0]] || null
      switch (lineType) {
        case "header":
          commitHunk()
          currentHunk = emptyHunk(line)
          break
        case null:
          // unrecognized, bail out
          state = "parsing header"
          commitFilePatch()
          i--
          break
        case "pragma":
          if (!line.startsWith("\\ No newline at end of file")) {
            throw new Error("Unrecognized pragma in patch file: " + line)
          }
          if (!currentHunkMutationPart) {
            throw new Error(
              "Bad parser state: No newline at EOF pragma encountered without context",
            )
          }
          currentHunkMutationPart.noNewlineAtEndOfFile = true
          break
        case "insertion":
        case "deletion":
        case "context":
          if (!currentHunk) {
            throw new Error(
              "Bad parser state: Hunk lines encountered before hunk header",
            )
          }
          if (
            currentHunkMutationPart &&
            currentHunkMutationPart.type !== lineType
          ) {
            currentHunk.parts.push(currentHunkMutationPart)
            currentHunkMutationPart = null
          }
          if (!currentHunkMutationPart) {
            currentHunkMutationPart = {
              type: lineType,
              lines: [],
              noNewlineAtEndOfFile: false,
            }
          }
          currentHunkMutationPart.lines.push(line.slice(1))
          break
        default:
          // exhausitveness check
          assertNever(lineType)
      }
    }
  }

  commitFilePatch()

  for (const { hunks } of result) {
    if (hunks) {
      for (const hunk of hunks) {
        verifyHunkIntegrity(hunk)
      }
    }
  }

  return result
}
```

然后呢，再对不同语句下生成不同的effect对象就可以。

```ts
export function interpretParsedPatchFile(files: FileDeets[]): ParsedPatchFile {
  const result: ParsedPatchFile = []

  for (const file of files) {
    const {
      diffLineFromPath,
      diffLineToPath,
      oldMode,
      newMode,
      deletedFileMode,
      newFileMode,
      renameFrom,
      renameTo,
      beforeHash,
      afterHash,
      fromPath,
      toPath,
      hunks,
    } = file
    const type: PatchFilePart["type"] = renameFrom
      ? "rename"
      : deletedFileMode
      ? "file deletion"
      : newFileMode
      ? "file creation"
      : hunks && hunks.length > 0
      ? "patch"
      : "mode change"

    let destinationFilePath: string | null = null
    switch (type) {
      case "rename":
        if (!renameFrom || !renameTo) {
          throw new Error("Bad parser state: rename from & to not given")
        }
        result.push({
          type: "rename",
          fromPath: renameFrom,
          toPath: renameTo,
        })
        destinationFilePath = renameTo
        break
      case "file deletion": {
        const path = diffLineFromPath || fromPath
        if (!path) {
          throw new Error("Bad parse state: no path given for file deletion")
        }
        result.push({
          type: "file deletion",
          hunk: (hunks && hunks[0]) || null,
          path,
          mode: parseFileMode(deletedFileMode!),
          hash: beforeHash,
        })
        break
      }
      case "file creation": {
        const path = diffLineToPath || toPath
        if (!path) {
          throw new Error("Bad parse state: no path given for file creation")
        }
        result.push({
          type: "file creation",
          hunk: (hunks && hunks[0]) || null,
          path,
          mode: parseFileMode(newFileMode!),
          hash: afterHash,
        })
        break
      }
      case "patch":
      case "mode change":
        destinationFilePath = toPath || diffLineToPath
        break
      default:
        assertNever(type)
    }

    if (destinationFilePath && oldMode && newMode && oldMode !== newMode) {
      result.push({
        type: "mode change",
        path: destinationFilePath,
        oldMode: parseFileMode(oldMode),
        newMode: parseFileMode(newMode),
      })
    }

    if (destinationFilePath && hunks && hunks.length) {
      result.push({
        type: "patch",
        path: destinationFilePath,
        hunks,
        beforeHash,
        afterHash,
      })
    }
  }

  return result
}
```

最后，把所有的不同类型的effect对象进行相对应的逻辑执行即可。

```ts
export const executeEffects = (
  effects: ParsedPatchFile,
  { dryRun }: { dryRun: boolean },
) => {
  effects.forEach(eff => {
    switch (eff.type) {
      case "file deletion":
        if (dryRun) {
          if (!fs.existsSync(eff.path)) {
            throw new Error(
              "Trying to delete file that doesn't exist: " + eff.path,
            )
          }
        } else {
          // TODO: integrity checks
          fs.unlinkSync(eff.path)
        }
        break
      case "rename":
        if (dryRun) {
          // TODO: see what patch files look like if moving to exising path
          if (!fs.existsSync(eff.fromPath)) {
            throw new Error(
              "Trying to move file that doesn't exist: " + eff.fromPath,
            )
          }
        } else {
          fs.moveSync(eff.fromPath, eff.toPath)
        }
        break
      case "file creation":
        if (dryRun) {
          if (fs.existsSync(eff.path)) {
            throw new Error(
              "Trying to create file that already exists: " + eff.path,
            )
          }
          // todo: check file contents matches
        } else {
          const fileContents = eff.hunk
            ? eff.hunk.parts[0].lines.join("\n") +
              (eff.hunk.parts[0].noNewlineAtEndOfFile ? "" : "\n")
            : ""
          fs.ensureDirSync(dirname(eff.path))
          fs.writeFileSync(eff.path, fileContents, { mode: eff.mode })
        }
        break
      case "patch":
        applyPatch(eff, { dryRun })
        break
      case "mode change":
        const currentMode = fs.statSync(eff.path).mode
        if (
          ((isExecutable(eff.newMode) && isExecutable(currentMode)) ||
            (!isExecutable(eff.newMode) && !isExecutable(currentMode))) &&
          dryRun
        ) {
          console.warn(`Mode change is not required for file ${eff.path}`)
        }
        fs.chmodSync(eff.path, eff.newMode)
        break
      default:
        assertNever(eff)
    }
  })
}
```

## 总结

使用方面：使用 `pnpm patch xxx` 生成一个临时目录，里面包含包源文件，到该目录下进行源码修改，之后使用 `pnpm patch-commit dir` 生成一个补丁，位于项目根目录下的 patches/xxx.patch 文件。之后将该文件上传到git中，下次拉取代码时，执行 pnpm install 时会自动应用。

原理方面：借助 git diff 的能力，分析两个目录下文件的异同，生成patch文件。在安装依赖的时候，解析patch文件，对不同类型的更改进行反向还原，达到修改依赖的效果。


