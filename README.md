# vroo

vroo is a programming language where the compiler is an AI. Think of it as executable pseudocode: it looks like real code, but the details are filled in for you.

You write `.vroo` files in whatever loose, brace-based syntax feels right, with a comment on anything that isn't obvious. `vroo` hands the file to a model, gets C back, and `cc` builds it.

It is a form of vibecoding, and I'm not going to pretend otherwise. If AI isn't your thing, don't use it. It's made so you feel like you wrote something while the AI does the hard part.

```
#include <io.vrh>
#include <os.vrh>

int main() {
    path = "."
    if args > 1 { path = args(1) }
    dir = opendir(path) or err {
        io.print(path, ": ", err)
        return 1
    }
    defer dirclose()
    horizontal(dir_names)
    return 0
}

int horizontal(names) {
    for name in names {
        io.color_add(name)   //prints the name in a random terminal color
        io.print("  ")
    }
    io.print("\n")
}
```

There is no grammar. The comment is the program: if you say what a line should do, that's what you get. If you don't, the model guesses the smallest thing, usually nothing, and leaves `/* vroo: guessed */` in the C so you can find it.

## Use

```
vroo prog.vroo -o prog     translate and build
vroo -c prog.vroo          print the C, build nothing
vroo make [file.vmake]     build a project: the .vmake and the files it names go to the agent as one program
```

File types: `.vroo` is source, `.vrh` is a header, `.vmake` is the build file. A `.vmake` says in plain words what the program is called (`program : name`) and which files are its parts. Any `.vroo` or `.vrh` in the folder that it mentions by name is sent along with it, and the whole lot comes back as one C file. `.vmake` is the least tested of the three.

Needs `sh`, `cc`, and either [Claude Code](https://claude.com/claude-code) on your PATH or an API key for any OpenAI-style chat API.

## How it behaves

The C is cached in `.vroo-cache`, keyed on a hash of the source and the prompt. The same source gives the same C until you change it, and a program that has built once never calls the model again. Delete the folder to translate fresh.

The first translation of a file takes around 20 seconds with Claude Code. Cached runs take a few hundredths of a second.

It translates what you wrote and guesses what you left out. It won't drop or reorder your statements.

Your source is sent to the model each time it is translated.

## Other models

Claude Code is the default. Any OpenAI-style chat API works too:

```
export VROO_API_URL=https://api.deepseek.com/chat/completions
export VROO_MODEL=deepseek-chat
export VROO_API_KEY=...
```

Other URLs: `https://api.openai.com/v1/chat/completions`, `https://api.moonshot.ai/v1/chat/completions`, `https://generativelanguage.googleapis.com/v1beta/openai/chat/completions` for Gemini, `https://openrouter.ai/api/v1/chat/completions`, and `http://localhost:11434/v1/chat/completions` for a local Ollama. The key is read from the environment and never written anywhere. This route needs `curl` and `python3`.

## Don't

Don't write a kernel in it, or anything that has to be bit-exact. A wrong guess there doesn't give wrong output, it hangs the machine. It's fine for tools, sketches and trying out ideas for syntax.

## License

ISC.
