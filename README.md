# KLEE Official Website

The KLEE website, built using [Web Starter Kit](http://developers.google.com/web/starter-kit) and [Jekyll](http://jekyllrb.com/).

## Quickstart

### Dependencies

* [Ruby](https://www.ruby-lang.org/) ≥ 2.0.0.  You can use [RVM](http://rvm.io/) or [rbenv](https://github.com/sstephenson/rbenv) to install it. (A Ruby DevKit is required to build dependencies with native extensions.)

* [Node.js](https://nodejs.org/)

* [Bundler](https://bundler.io/).  You can use `gem install bundle` to install it.

* [Python](https://www.python.org/)


### Installation

Clone this repository and install all dependencies using:

```bash
$ bundle
```

Then, you can preview the site by running (at `localhost:4000` by default):

```bash
$ bundle exec jekyll serve -w
```

To build the site, you can use:

```bash
$ bundle exec jekyll build
```

## Contributing

Contributions, both to content and design are welcome and encouraged. To contribute, please submit a pull request.

### Contributing to the Publication List

Please open a pull request for missing [publications](https://klee.github.io/publications/) that build upon or use KLEE.
All list entries are ordered by publication date and follow the layout below:

```markdown
1. [KLEE: Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs](http://www.doc.ic.ac.uk/~cristic/papers/klee-osdi-08.pdf)⎵⎵
⎵⎵Cristian Cadar, Daniel Dunbar, Dawson Engler⎵⎵
⎵⎵USENIX Symposium on Operating Systems Design and Implementation (OSDI 2008)⎵⎵
⎵⎵December 8-10, 2008, San Diego, CA, USA⎵⎵
⎵⎵**Klee is available [here](https://klee.github.io/).**
```

Note that the template language requires that two spaces (shown as `⎵`) are added to every but the last line to insert line-breaks.


## Adding Release Documentation

The repository has old versions of the documentation in `releases/docs/`. To generate documentation for a new release, do the following:

1. Open `_config.yml` and
 - Change `is_release` to `true`
 - Add the `doxygen` and `releases` folders to `exclude`
 - Set `current_version` to the new KLEE version

2. Run the following command, where `<VERSION>` is the KLEE version (e.g., "v2.3"):

```
$ jekyll build -d releases/docs/<VERSION> --baseurl /releases/docs/<VERSION>
```

3. Clear the changes made to `_config.yml`, except for the `current_version`
4. Add `releases/docs/<VERSION>` to the repository
5. Add an entry for the release in `releases/index.md`
6. Commit the changes

## License

Creative Commons Attribution 3.0 Unported (CC BY 3.0)
