STYLE(7) - Miscellaneous Information Manual

# NAME

**style** - template file style guide

# DESCRIPTION

This file specifies the preferred style for template source files in
**void-packages**.

These guidelines should be followed for all new submissions.
Old non-compliant templates should be made to conform when modified.

This manual assumes knowledge of what templates are, how they work and what they
are for.

# STYLE

## HEADER

Templates
**always**
start with a header

	# Template file for '<pkgname>'

If your package requires another package to have version synchrony a comment
can be added below the header.

	# Requires version and revision synchrony with '<pkgname>'

## VARIABLES

Immediately after begin the variables, it is required that
**pkgname**,
**version**
then
**revision**
start the template unless
**reverts**
is used, which must come between
**pkgname**
and
**version**.

	pkgname=foo
	reverts="1.0.1-rc2"
	version=1.0.0
	revision=1

If
**noarch**
is to be used, it should come after
**revision**

If applicable
**wrksrc**
and
**build\_wrksrc**
come right after revision or noarch, then
**build\_style**.
Variables that pertain to a build\_style like
**go\_import\_path**,
**configure\_args**,
**make\_install\_target**
must come directly after it. and afterwards come the dependencies.

Dependencies are ordered as
**hostmakedepends**,
**makedepends**,
**depends**,
**checkdepends**,

	hostmakedepends="automake gettext-devel glib-devel gobject-introspection gtk-doc
	 gtk-update-icon-cache pkg-config $(vopt_if wayland 'wayland-devel
	 wayland-protocols')"
	makedepends="at-spi2-atk-devel gdk-pixbuf-devel libepoxy-devel pango-devel
	 $(vopt_if colord 'colord-devel') $(vopt_if cups 'cups-devel')
	 $(vopt_if wayland 'libxkbcommon-devel wayland-devel wayland-protocols
	 libwayland-egl MesaLib-devel') $(vopt_if x11 'libXcursor-devel libXdamage-devel
	 libXext-devel libXinerama-devel libXi-devel libXrandr-devel
	 libXcomposite-devel')"
	depends="gtk-update-icon-cache shared-mime-info $(vopt_if x11 'dbus-x11')"

After the dependencies come
**short\_desc**,
start it with a Uppercase character, don't pass over 72 chars.
When writing
**short\_desc**
avoid starting with superfluous characters like A.

	# Wrong!
	# Superfluous usage of A
	short_desc="A configuration language guaranteed to terminate"
	# Correct!
	short_desc="Configuration language guaranteed to terminate"

**maintainer**
follows.
It uses a
**name &lt;email&gt;**
scheme, name and email must match the git equivalents used to commit.
The email must be reachable.
It is used in  case something needs to be done
about the package that will be used for contact.

Afterwards we have
**license**
which should be specified according to the SPDX short-identifier.
If multiple licenses apply then separate them with a comma and a whitespace.

	# Wrong!
	# GPL-2 isn't a SPDX short-identifier
	# there is no proper separation between the licenses
	license="GPL-2 MPL-2.0"
	# Correct!
	license="LGPL-2.1-or-later, Apache-2.0"

**homepage**,
**changelog**
and
**distfiles**
follow, if multiple distfiles are present then they should be broken with a
newline, the same applies to
**checksum**

	homepage="https://foosoft.com"
	changelog="https://foosoft.com/dist/foo/changelog.txt"
	distfiles="https://foosoft.com/dist/foo/${version}.tar.xz
	 https://foosoft.com/dist/bar/${version}.tar.xz"

Variables that are knobs to functionality and are not specific to a
build\_style should come at the end of the variables section.

The same applies for variables that take structured values like
**conf\_files**.
There is no specific ordering for such variables. but it is preferred that
multi-line versions of
**conf\_files**
and other variables that are prone to requiring multiple lines like mkdirs
should be at the end, with a blank space separating them from the other
variables.

When using
**nopie**,
**nocross**
and
**broken**
make the value be something descriptive, if no other description is available,
a build log showcasing the failure is acceptable.

Also it is preferred using
**broken**
inside an
**case in esac**
control structure to
**only\_for\_arches**
because the latter needs to add comments in the template file to explain why while
**broken**
can provide a message to the user as its value.

If
**alternatives**
is used it should be the last line, if there are multiple alternatives
presented then they should be 1 on each line.

	checksum=blablablablablablablablablabla
	noarch=yes
	
	alternatives="foo:foo:usr/bin/foo
	 foo:bar:usr/bin/bar"

Some rules to follow when quoting variables:

-	Don't quote unless necessary.

It is necessary to quote when:

-	There is a strong possibility of whitespace appearing like on
	**license**
	and any variables ending with \_args.

-	Subshell is being run to fill a variable's value.

-	Variable is being expanded.

When expanding variables inside another variables put the variable being
expanded inside brackets and quote the whole variable.

**pkgname**
and
**version**
must never be quoted, if they need to be quoted then they don't satisfy the
rules for those fields

If using variables that are not part of xbps-src as defined per the manual then
prefix them with a underscore.

## CONTROL STRUCTURES

The variables are over, now begins the
**control structures**
which are any conditionals and structures provided by the shell to control the flow
of execution of the template, like:

-	if EXPRESSION; then COMMAND-LIST; fi

-	case EXPRESSION in CASE1) COMMAND-LIST;; ... CASEN) COMMAND-LIST;; esac

-	for NAME \[in LIST ]; do COMMANDS; done

-	while CONTROL-COMMAND; do CONSEQUENT-COMMANDS; done

they should start 1 blank line after the variables, and between each occurrence
of a
**control structure**
there should be 1 blank line of separation, there is no specific ordering for
control structures.

When declaring one make the start of the declaration in one line,
1 hard tab of indentation applies inside the control structure.

When using a control structure there should be 1 whitespace separating the
semicolon and the right-side keyword.
No spacing is needed between the semicolon and the left side of the declaration.
The whole declaration must be on one line.

when checking if a variable is not empty via
**\[ ]**
the use of -n can be discarded.

	# Wrong!
	# space between ] and ;
	if [ -n "$CROSS_BUILD" ] ; then
		hostmakedepends+=" glib-devel"
	fi
	
	# Wrong!
	# then keyword should be on the same line
	if [ -n "$CROSS_BUILD" ]
	then
		hostmakedepends+=" glib-devel"
	fi
	
	# Correct!
	if [ "$CROSS_BUILD" ]; then
		hostmakedepends+=" glib-devel"
	fi

## BUILD PHASES

After control structures comes the
**build phases**
which are functions.
That means they have 1 hard tab of indentation inside them.
They also follow the same spacing rules as
control structures
and their ordering should follow what is run on when building the package, per
example:
**do\_install**
comes after
**post\_build**.

Inside
**build phases**
there is no strict style for separation between variables, control structures and
any other content, but it should be done if it helps to make it readable.

## SUBPACKAGES

The last section for the template is the
**subpackages**
declarations which are functions that are marked
**&lt;subpkgname&gt;\_package()**,
the same spacing and indentation rules of
**build phases**
apply here.

## EXTRA GUIDELINES

The following are guidelines that apply to the whole structure of the template and
guidelines for when following this document.

Don't be smart when writing a template, keep it clear and readable.
Other people and future you appreciate.

Try not pass 80 columns but don't try to be smart about doing it though with
multiple variables and weird-looking structures.
If you really need all those columns or it is more readable when done that way
it is acceptable to do it.

When breaking a line into multiple lines, indent the newline with 1 space.

These are guidelines that maintainers and developers should follow and reviewers
should apply but they are not laws and should be broken when good reasons show
up to justify.

# REVISIONS

## Aug 31, 2018

Creation of this document

## UPDATING MARKDOWN

The markdown version of this file available on void-packages should be
updated by making changes to the manpage and the changes regenerated
with mandoc.

	$ mandoc path/to/style.7 -Tmarkdown > STYLE.md

# AUTHORS

maxice8
&lt;thinkabit.ukim@gmail.com&gt;

Void Linux - May 1, 2018
