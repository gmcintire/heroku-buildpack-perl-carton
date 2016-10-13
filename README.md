Heroku/Dokku buildpack: Perl-Procfile-Carton
=====================================

This is a Heroku buildpack that makes it easy to run any Perl application
(possibly a complex one with different components) from a `Procfile`.

This fork combines features from [heroku-buildpack-perl][hbp], the
[carton branch of heroku-buildpack-perl][hbpc] and
[heroku-buildpack-perl-procfile][hbpp]. Resulting in:

- any tool can be run in Procfile (not just Starman or plackup)
- vendored version of perl will be built based on .perl-version
- modules will be installed using Carton and cpanfile.snapshot

If your repository has a top-level sub-directory named either `epan` or
`dpan`, it's assumed to have a CPAN-like directory structure inside (with
`authors` and `modules` and all the rest) and it will be used for looking
for modules. This makes it easier to ship *DarkPAN* modules while still
being able to install the official ones from the official CPAN.

This buildpack is a fork of [heroku-buildpack-perl-heroku][hbpp] and will
hopefully be upstreamed.

Usage
-----

Example usage in Heroku (untested):

    $ ls
    cpanfile
    cpanfile.snapshot
    Procfile
    lib/

    $ cat cpanfile
    requires Mojolicious => '7.00';

    $ cat Procfile
    web: hypnotoad -f ./script/app.pl

    $ heroku create --stack cedar \
       --buildpack https://github.com/mvgrimes/heroku-buildpack-perl-carton.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Perl/Procfile app detected
    -----> Installing dependencies

The buildpack will detect that your perl-app has an `Procfile`
and a `cpanfile.snapshot` in the root.

Example usage in [Dokku][], using a `.buildpacks` file inside the
repository:

    $ ls -a
    .buildpacks
    .git
    .perl-version
    cpanfile
    cpanfile.snapshot
    Procfile
    lib/

    $ cat .buildpacks
    https://github.com/mvgrimes/heroku-buildpack-perl-carton.git

    $ cat .perl-version
    5.22.2

    $ cat cpanfile
    requires Mojolicious => '7.00';

    $ cat Procfile
    web: hypnotoad -f ./script/app.pl

    $ ssh dokku@your-node.example.com apps:create your-app

    $ git add remote dokku dokku@your-node.example.com:your-app

    $ git push dokku master
    ...
    -----> Fetching custom buildpack
    -----> Perl/Procfile app detected
    ...

Alternatively, if you don't want to use the `.buildpacks` file inside the
repo, you can use the suggestions provided in [custom buildpacks][]:

    $ ssh dokku@your-node.example.com config:set your-app \
        BUILDPACK_URL=https://github.com/mvgrimes/heroku-buildpack-perl-carton.git


Libraries
---------

Dependencies can be declared using `cpanfile` (recommended) and
`cpanfile.snapshot`, and the buildpack will install these dependencies using
[carton][] into `./local` directory.

You can ship your own *DarkPAN* modules by creating a subdirectory named either
`epan` or `dpan` at the top level, and shaping it in a CPAN-compatible way.
Example:

    $ find epan -type f | sort
    epan/authors/01mailrc.txt.gz
    epan/authors/id/P/PO/POLETTIX/Some-DarkPAN-Module-0.01.tar.gz
    epan/modules/02packages.details.txt.gz
    epan/modules/03modlist.data.gz

To create it, you might want to look into:

- [epan][]
- [OrePAN2][]
- [CPAN::Faker][]

and probably a few more. When either (or both) directory is present, it is
set as a `--mirror` for [cpanm][] *before*
[http://www.cpan.org](http://www.cpan.org) so that you can e.g. ship
patched versions of official modules.


[cpanm]: http://cpanmin.us
[epan]: https://github.com/polettix/epan
[OrePAN2]: https://github.com/tokuhirom/OrePAN2
[CPAN::Faker]: https://metacpan.org/pod/CPAN::Faker
[Dokku]: http://dokku.viewdocs.io/dokku/
[custom buildpacks]: http://dokku.viewdocs.io/dokku/deployment/methods/buildpacks/#specifying-a-custom-buildpack
[hbp]: https://github.com/miyagawa/heroku-buildpack-perl
[hbpp]: https://github.com/kazeburo/heroku-buildpack-perl-procfile
[hbpc]: https://github.com/miyagawa/heroku-buildpack-perl/tree/carton
