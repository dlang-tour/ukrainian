# Локальна установка D

Ви можете [завантажити](http://dlang.org/download.html) та встановити
найсвіжішу версію референс-компілятора **DMD** (Digital Mars D) з
офіційного сайту [dlang.org](https://dlang.org):

### Windows

* [Інсталятор](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd-{{latest-release}}.exe)
* чи: [Архів](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.windows.7z)
* чи за допомогою пакетного менеджера [chocolatey](https://chocolatey.org/packages/dmd): `choco install dmd`

### macOS

* `.dmg` [пакет](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.dmg)
* чи: [Архів](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.osx.tar.xz)
* чи за допомогою пакетного менеджера [Homebrew](http://brew.sh): `brew install dmd`

### Linux / FreeBSD / macOS

Для швидкого встановлення dmd у свою домашню теку виконайте:
 `curl -fsS https://dlang.org/install.sh | bash -s dmd`

Також доступні пакети для різних дистрибутивів:

* [ArchLinux](https://wiki.archlinux.org/index.php/D_(programming_language))
* [Debian/Ubuntu](http://d-apt.sourceforge.net)
* [Fedora/CentOS](http://dlang.org/download.html#dmd)
* [Gentoo](https://wiki.gentoo.org/wiki/Dlang)
* [OpenSuse](http://dlang.org/download.html#dmd)

### Інші компілятори

Крім референс-компілятора DMD, який використовує свій власний backend,
існують ще два компілятори, які Ви можете знайти у секції завантажень сайту
[dlang.org](https://dlang.org):

* [**GDC**](http://gdcproject.org/downloads) використовує GCC backend
* [**LDC**](https://github.com/ldc-developers/ldc#installation) заснований на LLVM backend

GDC та LDC не завжди відповідають найсвіжішій frontend-версії DMD, 
але надають більш високі рівні оптимізації і можливість компіляції на
інші платформи на зразок ARM.

Більше інформації про компілятори можете отримати на [wiki-сторінці]
(https://wiki.dlang.org/Compilers)

