# Maintainer: Christian Hesse <mail@eworm.de>
# Maintainer: Dan McGee <dan@archlinux.org>

pkgname=git
pkgver=2.43.0
pkgrel=1
pkgdesc='the fast distributed version control system'
arch=('x86_64')
url='https://git-scm.com/'
license=('GPL2')
depends=('curl' 'expat' 'perl' 'perl-error' 'perl-mailtools'
         'openssl' 'pcre2' 'grep' 'shadow' 'zlib')
makedepends=('python' 'xmlto' 'asciidoc')
checkdepends=('openssh')
optdepends=('tk: gitk and git gui'
            'openssh: ssh transport and crypto'
            'perl-libwww: git svn'
            'perl-term-readkey: git svn and interactive.singlekey setting'
            'perl-io-socket-ssl: git send-email TLS support'
            'perl-authen-sasl: git send-email TLS support'
            'perl-mediawiki-api: git mediawiki support'
            'perl-datetime-format-iso8601: git mediawiki support'
            'perl-lwp-protocol-https: git mediawiki https support'
            'perl-cgi: gitweb (web interface) support'
            'python: git svn & git p4'
            'subversion: git svn'
            'org.freedesktop.secrets: keyring credential helper'
            'libsecret: libsecret credential helper')
install=git.install
validpgpkeys=('96E07AF25771955980DAD10020D04E5A713660A7') # Junio C Hamano
source=("https://www.kernel.org/pub/software/scm/git/git-$pkgver.tar."{xz,sign}
        'git-daemon@.service'
        'git-daemon.socket'
        'git-sysusers.conf')
sha256sums=('5446603e73d911781d259e565750dcd277a42836c8e392cac91cf137aa9b76ec'
            'SKIP'
            '14c0b67cfe116b430645c19d8c4759419657e6809dfa28f438c33a005245ad91'
            'ac4c90d62c44926e6d30d18d97767efc901076d4e0283ed812a349aece72f203'
            '7630e8245526ad80f703fac9900a1328588c503ce32b37b9f8811674fcda4a45')

_make() {
  local make_options=(
    prefix='/usr'
    gitexecdir='/usr/lib/git-core'
    perllibdir="$(/usr/bin/perl -MConfig -wle 'print $Config{installvendorlib}')"

    CFLAGS="$CFLAGS"
    LDFLAGS="$LDFLAGS"
    INSTALL_SYMLINKS=1
    MAN_BOLD_LITERAL=1
    NO_PERL_CPAN_FALLBACKS=1
    USE_LIBPCRE2=1
  )

  make "${make_options[@]}" "$@"
}

build() {
  cd "$srcdir/$pkgname-$pkgver"

  _make all man

  _make -C contrib/credential/libsecret
  _make -C contrib/subtree all man
  _make -C contrib/mw-to-git all
  _make -C contrib/diff-highlight
}

check() {
  cd "$srcdir/$pkgname-$pkgver"

  local jobs
  jobs=$(expr "$MAKEFLAGS" : '.*\(-j[0-9]*\).*') || true
  mkdir -p /dev/shm/git-test
  # explicitly specify SHELL to avoid a test failure in t/t9903-bash-prompt.sh
  # which is caused by 'git rebase' trying to use builduser's SHELL inside the
  # build chroot (i.e.: /usr/bin/nologin)
  SHELL=/bin/sh \
  _make \
    NO_SVN_TESTS=y \
    DEFAULT_TEST_TARGET=prove \
    GIT_PROVE_OPTS="$jobs -Q" \
    GIT_TEST_OPTS="--root=/dev/shm/git-test" \
    test
}

package() {
  cd "$srcdir/$pkgname-$pkgver"

  _make \
    DESTDIR="$pkgdir" \
    install install-man

  # bash completion
  mkdir -p "$pkgdir"/usr/share/bash-completion/completions/
  install -m 0644 ./contrib/completion/git-completion.bash "$pkgdir"/usr/share/bash-completion/completions/git
  # fancy git prompt
  mkdir -p "$pkgdir"/usr/share/git/
  install -m 0644 ./contrib/completion/git-prompt.sh "$pkgdir"/usr/share/git/git-prompt.sh
  # libsecret credentials helper
  install -m 0755 contrib/credential/libsecret/git-credential-libsecret \
      "$pkgdir"/usr/lib/git-core/git-credential-libsecret
  _make -C contrib/credential/libsecret clean
  # subtree installation
  _make -C contrib/subtree DESTDIR="$pkgdir" install install-man
  # mediawiki installation
  _make -C contrib/mw-to-git DESTDIR="$pkgdir" install
  # the rest of the contrib stuff
  find contrib/ -name '.gitignore' -delete
  cp -a ./contrib/* "$pkgdir"/usr/share/git/

  # git-daemon via systemd socket activation
  install -D -m 0644 "$srcdir"/git-daemon@.service "$pkgdir"/usr/lib/systemd/system/git-daemon@.service
  install -D -m 0644 "$srcdir"/git-daemon.socket "$pkgdir"/usr/lib/systemd/system/git-daemon.socket

  # sysusers file
  install -D -m 0644 "$srcdir"/git-sysusers.conf "$pkgdir"/usr/lib/sysusers.d/git.conf
}
