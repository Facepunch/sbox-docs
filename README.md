# s&box docs

This repository contains the documentation source files for s&box, which is viewed on [sbox.game/dev/doc](https://sbox.game/dev/doc).

Documentation is written in Markdown and synced onto sbox.game after it's pushed to master.

## Local preview

With the .NET SDK installed, install DocFX once:

```powershell
dotnet tool install --global docfx
```

From this repository, build and serve the pages:

```powershell
docfx build docfx.json
docfx serve _site --hostname localhost --port 8080
```

Open <http://localhost:8080/docs/animation/animgraph/index.html> for the AnimGraph guide. Rebuild after editing, then refresh your browser. The generated `_site` folder is ignored by Git.

DocFX is a content preview, not an exact reproduction of sbox.game's styling or custom Markdown features.

## Contributing

Anyone can contribute to the documentation by making a pull request. Some guidelines:

* Good English, grammar and spelling
* Keep it simple, don't overexplain concepts
* We're documenting s&box, we don't need pages on how to install Visual Studio or Blender
* Review your own pull request before marking it as ready to review

## License

This project is licensed under [Creative Commons Attribution 4.0](LICENSE).