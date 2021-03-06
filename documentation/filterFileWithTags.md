 # Filter and parse file with tags

To parse lines to markdown style we use the parse functions [@MDParseLine](./mdParseLine.md)

 ## filterFileWithTags

 **Main function** to parse and filter all source code lines with functions to decide how the line must be printed in the markdown document.

 This function returns an object with this data:

<details>

<summary>Function parameters</summary>

<br>

 - IGNORE  - `boolean` - that indicates no line was parsed to markdown

 - MDLINES - `array`   - array with all markdown lines filter by tags

 - NOTAGSRESULT - `array` - array with all lines as block code if no tags was found in source code

</details>

```
const filterFileWithTags = (fileData) => {
  const fileByTags = {
    IGNORE: false,
    MDLINES: [],
    NOTAGSRESULT: [],
  };
  const fileLinesInArray = fileData.split(/\r?\n/);
  const FLAGS = {
    isCodeBlockAll: false,
    isCodeBlock: false,
    isRenderReturnBlock: false,
    parentesisGroupCount: 1,
    hasOtherTag: false,
  };
  if (fileLinesInArray[0].includes(TAGS.IGNORE)) {
    fileByTags.IGNORE = true;
    return fileByTags;
  }
  if (fileLinesInArray[0].includes(TAGS.CBALL)) {
    FLAGS.isCodeBlockAll = true;
    fileByTags.MDLINES.push(MDParseLine.getCodeBlockStart());
  }
  fileLinesInArray.forEach((line) => {
    if (isToCodeBlockAll(FLAGS, line)) {
      FLAGS.hasOtherTag = true;
      line = MDParseLine.escapeJSCaracthersInLIne(line);
      fileByTags.MDLINES.push(MDParseLine.lineAsCode(line));
    } else if (isLineWithCodeBlockStartFlag(line)) {
      FLAGS.isCodeBlock = true;
      FLAGS.hasOtherTag = true;
      fileByTags.MDLINES.push(MDParseLine.getCodeBlockStart());
    } else if (isLineWithCodeBlockEndFlag(line)) {
      FLAGS.isCodeBlock = false;
      fileByTags.MDLINES.push(MDParseLine.getCodeBlockEnd());
    } else if (isLineWithCommentsInFirstPosition(line)) {
      fileByTags.NOTAGSRESULT.push(MDParseLine.lineAsMarkdown(line));
      fileByTags.MDLINES.push(MDParseLine.lineAsMarkdown(line));
    } else if (isLineInsideCodeBlock(FLAGS, line)) {
      line = MDParseLine.escapeJSCaracthersInLIne(line);
      fileByTags.MDLINES.push(MDParseLine.lineAsCode(line));
    } else if (fileHasNoOtherTag(FLAGS, line)) {
      line = MDParseLine.escapeJSCaracthersInLIne(line);
      fileByTags.NOTAGSRESULT.push(MDParseLine.lineAsCode(line));
    }
  });
  if (FLAGS.hasOtherTag) {
    fileByTags.NOTAGSRESULT = [];
  } else {
    fileByTags.NOTAGSRESULT.unshift(MDParseLine.getCodeBlockStart());
    fileByTags.NOTAGSRESULT.push(MDParseLine.getCodeBlockEnd());
  }
  if (FLAGS.isCodeBlockAll) {
    fileByTags.MDLINES.push(MDParseLine.getCodeBlockEnd());
  }
  const fileByTagsWithout = removeLastAddCharIfExists(fileByTags);
  return fileByTagsWithout;
};
```
---

 ## isToCodeBlockAll

 Returns `true` if the line is a CBAll tag

```
const isToCodeBlockAll = (FLAGS, line) => {
  return FLAGS.isCodeBlockAll && line !== "" && !line.includes(TAGS.CBALL);
};
```
---

 ## isLineWithCodeBlockStartFlag

 Returns `true` if the line is a CBStart tag

```
const isLineWithCodeBlockStartFlag = (line) => {
  return line.includes(TAGS.CBSTART);
};
```
---

 ## isLineWithCodeBlockEndFlag

 Returns `true` if the line is a CBEnd tag

```
const isLineWithCodeBlockEndFlag = (line) => {
  return line.includes(TAGS.CBEND);
};
```
---

 ## isLineInsideCodeBlock

 Returns `true` if a code block is 'open' and indicates that the line must be printed as a code line.

```
const isLineInsideCodeBlock = (FLAGS, line) => {
  return FLAGS.isCodeBlock && line !== "";
};
```
---

 ## isLineWithCommentsInFirstPosition

 Returns `true` if the line is a comment and is not a tag.

```
const isLineWithCommentsInFirstPosition = (line) => {
  const commentsRegex = RegExp(/^\s*\/\//);
  if (commentsRegex.test(line)) {
    return !isTagLine(line);
  }
  return false;
};
```
---

 ## isTagLine

 Returns `true` if the line is a tag (CBAll, CBStart, CBEnd or ignore) line.

```
const isTagLine = (line) => {
  return (
    TAGS[line.replace("//", "").replace("@", "").toUpperCase()] !== undefined
  );
};
```
---

 ## fileHasNoOtherTag

 This function is used to put all file lines at the NOTAGSRESULT array if no other tag was found in file.

```
const fileHasNoOtherTag = (FLAGS, line) => {
  return (
    !FLAGS.hasOtherTag &&
    !FLAGS.isCodeBlockAll &&
    !isLineWithCodeBlockStartFlag(line) &&
    !isLineWithCodeBlockEndFlag(line) &&
    !isLineWithCommentsInFirstPosition(line) &&
    !isLineInsideCodeBlock(FLAGS, line) &&
    !FLAGS.isCodeBlock &&
    line !== ""
  );
};
```
---

 ## removeLastAddCharIfExists

 Function to remove the `+` characther if it exists in the last array element.

 This function is necessary to delete any extra plus character.

```
const removeLastAddCharIfExists = (fileByTags) => {
  const localArrayFilesByTag = { ...fileByTags };
  const { MDLINES, NOTAGSRESULT } = localArrayFilesByTag;
  if (MDLINES && MDLINES.length > 0) {
    MDLINES[MDLINES.length - 1] = MDLINES[MDLINES.length - 1].replace(
      /\+\s*\$/,
      ""
    );
  }
  if (NOTAGSRESULT && NOTAGSRESULT.length > 0) {
    NOTAGSRESULT[NOTAGSRESULT.length - 1] = NOTAGSRESULT[
      NOTAGSRESULT.length - 1
    ].replace(/\+\s*\$/, "");
  }
  return localArrayFilesByTag;
};
```
