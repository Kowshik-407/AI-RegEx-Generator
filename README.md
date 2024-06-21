# AI-RegEx-Generator
Used ChatGPT to Build a RegEx Generator with JS &amp; Python Editors - Open AI API Low Code

Tools used:
- Retool : https://retool.com
- OpenAI : https://platform.openai.com/playground

## Projects
**AI RegEx Generator** : [JS_Editor][js-editor]

_RegEx Generator - JS Editor Template_
![image](https://github.com/Kowshik-407/AI-RegEx-Generator/assets/66817358/b18cb50f-30d9-4a32-9484-25c2228f5b57)


**AI RegEx Generator With JS & Python Editors** : [JS_Python_Editor][js-python-editors]

_RegEx Generator - JS + Python Editor Template_
![image](https://github.com/Kowshik-407/AI-RegEx-Generator/assets/66817358/04f569f3-845c-4138-aa1e-7d13ebd7becb)



## Code

<b><i>iFrame Block: </i></b>
```
<script 
  src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.11/codemirror.min.js" 
  integrity="sha512-rdFIN28+neM8H8zNsjRClhJb1fIYby2YCNmoqwnqBDEvZgpcp7MJiX8Wd+Oi6KcJOMOuvGztjrsI59rly9BsVQ==" 
  crossorigin="anonymous" 
  referrerpolicy="no-referrer">
</script>
<script 
  src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.11/mode/javascript/javascript.min.js" 
  integrity="sha512-Cbz+kvn+l5pi5HfXsEB/FYgZVKjGIhOgYNBwj4W2IHP2y8r3AdyDCQRnEUqIQ+6aJjygKPTyaNT2eIihaykJlw==" 
  crossorigin="anonymous" 
  referrerpolicy="no-referrer">
</script>

<style>
  @import url('https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.11/codemirror.min.css');
  @import url('https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.11/theme/eclipse.min.css');

  html, body {
    height: 100%;
    width: 100%;
    overflow: hidden;
    padding: 0;
    margin: 0;
    box-sizing: border-box;
  }

  html {
    border: 1px solid #d1d1d1;
    border-radius: 4px;
  }

  body {
    padding: 0 4px;
  }

  .CodeMirror {
    height: auto;
  }
</style>

<script>
// Prefix any variables in the window scope with "r_" since we know we'll be
// evaluating developer code and want to try as best we can to avoid collision
let r_codeEditor = null;
let r_lastTest = '';
let r_lastRegex = '';
let r_runIndex = 0;
let r_resetIndex = 0;

// Send an object to be JSON printed in the Retool app
function print(obj) {
  if (!window.Retool || !window.Retool.modelUpdate) return;

  window.Retool.modelUpdate({
    runResult: obj
  });
}

// Init a CodeMirror editor (with timeouts to ensure scripts are loaded first)
function r_createEditor() {
  if (!CodeMirror) {
    setTimeout(createEditor, 50);
  } else {
    // Initialize code editor
    r_codeEditor = CodeMirror(document.body, {
      value: '// Initializing...',
      mode: 'javascript',
      tabSize: 2,
      lineNumbers: true
    });
  }
}

// Set the contents of the CodeMirror editor (with timeouts to ensure the
// editor has been created first)
function r_setCode(code) {
  if (!r_codeEditor) {
    // If code editor is not initialized, wait and retry
    setTimeout(() => {
      r_setCode(code);
    }, 60);
  } else {
    r_codeEditor.setValue(code);
  }
}

// Evaluate the current content of the code editor and return the result
function r_runCode() {
  if (!r_codeEditor) return;

  const code = r_codeEditor.getValue();
  if (!code) return;

  // If we have a code string, try and eval it - YOLO!
  try {
    eval(code);
  } catch(e) {
    // Print the error from the eval attempt to help debug
    console.error('Error running test code in editor:');
    console.error(e);
    print('Error running code snippet: \n' + e.message);
  } finally {
    r_runIndex++;
  }
}

function initCodeFromModel(model) {

}

// Subscribe for updates from the Retool host page
window.Retool.subscribe(function(model) {
  function initCode() {
    r_lastTest = model.testString;
    r_lastRegex = model.regex;

    const code = `const regex = ${r_lastRegex.trimStart()};
const t = ${JSON.stringify(r_lastTest)};
const matches = t.match(regex);

// "print" is a special function that will 
// log the results of your regex matches 
// to the text field below.
print(matches);
`;

    // Update the code in the code editor when initialized
    r_setCode(code);
  }

  // If the model hasn't been initialized yet, ignore
  if (!model) return;

  // Check for a requested test run - if the index is greater than zero and is 
  // greater than the last code run, eval the contents of the editor
  if (model.lastRunIndex > r_runIndex) {
    r_runCode();
  }

  // If the code and test string haven't changed, don't do anything else
  if (
    model.regex === r_lastRegex &&
    model.testString === r_lastTest
  ) {
    if (model.resetIndex > r_resetIndex) {
      // In this case, we have a reset request and should update code from 
      // the retool model
      r_resetIndex++;
      initCode();
    }
  } else {
    initCode();
  }
});

// Run initial load for the code editor
r_createEditor();
</script>
```

***Run Button Script - Event Handlers Success:***
```
const history = promptHistory.value;
if(!history.includes(regexPrompt.value)){
  history.unshift(regexPrompt.value);
  promptHistory.setValue(history);
}

// Update text fields
if(
  generateJSRegex.data && generateJSRegex.data?.choices?.length > 0
){
  const choice = generateJSRegex.data.choices[0];
  generatedJSRegex.setValue(choice.text);
  
  // Update UI + state
  currentJSRegexText.setValue(choice.text);
  currentTestString.setValue(regexTextString.value);
  
  codeGenerator1.updateModel({
    regex: choice.text,
    testString: regexTextString.value
  });
}
```

[js-editor]: https://kowshik407.retool.com/apps/9ea95ec4-23c2-11ee-972d-7b73343848eb/regexgenerator-openai
[js-python-editors]: https://kowshik407.retool.com/apps/53d26258-2488-11ee-91f2-eff767e0426b/regexgenerator-openai-editors
