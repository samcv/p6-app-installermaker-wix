# Perl 6 WiX Installer Maker

Written an application in Perl 6? Want to give Windows users an MSI so they can
easily install it? That's what this little program is here to help with.

Fair warning: it does something close to the Simplest Thing That Could Possibly
Work, which may or may not meet your needs, and at the time of publication has
been applied to make an installer for a single application. (If you have some
luck with it, feel free to send a PR to change this description!) Please
consider this tool "free as in puppy" - that is, you can freely take it and use
it, and if it helps that's great, but you might need to give it some attention
along the way. If you make changes that you think are useful to others, feel
free to PR them.

## How it works

This tool:

* Builds a private MoarVM, NQP, and Rakudo Perl 6 of the requested version
* Installs a zef (module installer) for use with this
* Uses that to install your application alongside the privately built Rakudo,
  either from the module ecosystem or from on disk, together with all of its
  dependencies
* Generates a WiX XML file
* Applies `candle` and `light`, resulting in an MSI

## What this tool does not do

* Go to any effort to hide the original source code from anyone curious enough
  to spend a few minutes hunting inside the install directory
* Compile your code into `exe`/`dll` files
* Handle things that use `Inline::Perl5`, `Inline::Python`, etc. (Modules that
  use native libraries are fine if they're in the set of modules that, when
  installed on Windows, will grab the required DLL and install it as a resource
  for the module to use at runtime. Modules that do this include `GTK::Simple`,
  `Digest::SHA1::Native`, `SSH::LibSSH`, and `Archive::Libarchive`.)

## What you'll need

* Git
* Perl 5 (tested with ActiveState Perl, though likely shouldn't matter)
* The Visual C++ build tools, and `nmake`/`cl`/`link` on path. Note that this
  does not imply installing Visual Studio; it is possible to freely download the
  [standalone compiler](http://landinghub.visualstudio.com/visual-cpp-build-tools).
  (It's probably possible, without too much trouble, to patch this tool to use
  other compilers.)
* [WiX](http://wixtoolset.org/releases/)

## How to use it

Write a YAML configuration file like this:

    # Versions of MoarVM, NQP, and Rakudo to use. Only 'rakudo' is required,
    # and it will then use the same value for NQP and MoarVM. This will work
    # except in the case where you want to refer to a commit/branch in the
    # Rakudo repository for some reason, since these are actually used to do a
    # checkout in the git repositories.
    versions:
        - moar: 2017.02
        - nqp: 2017.02
        - rakudo: 2017.02

    # The installation target location (currently Perl 6 is not relocatable).
    install-location: C:\MyApplication

    # The application to install (will be passed to `zef install`), so you can
    # actually list multiple things here if you wish.) You can also pass a path
    # if the project is not in the Perl 6 module ecosystem.
    application: App::MyApplication

    # Some WiX configuration. You must generate unique GUIDs for your app. Get
    # them [here](https://www.guidgenerator.com/) whlie supplies last! Check
    # the dashes, uppercase, and braces boxes.
    wix:
        guid: '{YOUR-GUID-HERE}'
        name: Your Glorious Application
        manufacturer: Your Wonderful Company
        version: 6.6.6
        language: 1033
        component-guid: '{A-DIFFERENT-GUID-HERE}'

Then run this application with that YAML file:

    make-perl6-wix-installer my-glorious-app.yml

All being well, you'll get an MSI file out.