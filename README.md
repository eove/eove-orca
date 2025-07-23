> [!Warning]  
> This is the eove-specific implementation of the public O.R.CA. If you're looking for the open-source O.R.CA, please have a look at https://github.com/eove/orca (from where this repo was forked).
> Whenever changes are to be made on O.R.CA, you should decide whether the change is internal to Eove or can be open-sourced.  
> If it can be open-sourced, please commit directly to the public O.R.CA repository.  
> You should then pull these changes back to the private Eove O.R.CA (this repo), by following the process below:  
> We first create the upstream remote if it does not exist:  
> `git remote | grep -q upstream-public || git remote add upstream-public git@github.com:eove/orca.git`  
> We then pull the public content of the `main` branch:  
> `git fetch upstream-public`  
> FInally, we merge the public O.R.CA with the Eove-private O.R.CA copy:  
> `git merge upstream-public/main`  

# O.R.CA

![A cartoonish drawing of a cute baby orca holding a key in its mouth ](./orca.png)

O.R.CA is an Offline Root Certificate Authority.

The documentation is present in the `docs` folder as markdown files.

You can open it in webbrowser on your machine by running the following command:
```bash
mdbook watch --open
```

> [!Note]  
> The nix shell provided in the flake besides this readme automatically makes `mdbook` available to you

If you'd rather browse the documentation directly from the sources, your entry point will be [SUMMARY.md](./docs/SUMMARY.md)
