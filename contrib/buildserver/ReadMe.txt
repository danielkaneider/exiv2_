contrib/buildserver/ReadMe.txt
------------------------------

737 rmills@rmillsmm:~/gnu/exiv2/trunk/contrib/buildserver $ dir
User Documentation and Scripts
-rw-r--r--@ 1 rmills  staff    63B 15 Dec 22:26 ReadMe.txt                <---- This file
-rwxr-xr-x@ 1 rmills  staff   440B 15 Dec 23:07 cmakeDailyAll.sh*         <---- run cmake_daily.sh on all platforms
-rwxr-xr-x@ 1 rmills  staff   424B 14 Dec 23:12 testDailyAll.sh*          <---- run test_daily.sh on all platform
-rw-r--r--@ 1 rmills  staff   655B 19 Dec 21:06 functions.so              <---- library for the scripts

Documentation and Scripts used by Jenkins
-rwxr-xr-x+ 1 rmills  staff    11K 15 Dec 22:17 jenkins_build.sh*         <---- Primary build script (called by Jenkins)
-rw-r--r--+ 1 rmills  staff   6.3K 15 Dec 22:17 jenkins_build.bat         <---- Windows build script (called by jenkins_build.sh)
-rwxr-xr-x@ 1 rmills  staff   5.8K 14 Dec 21:55 cmake_daily.sh*           <---- Builds exiv2 using cmake on all platforms
-rwxr-xr-x@ 1 rmills  staff   3.6K 15 Dec 05:49 test_daily.sh*            <---- Called by Jenkins to sync and build all platforms
-rw-r--r--@ 1 rmills  staff   4.0K 14 Dec 18:31 dailyReadMe.txt           <---- Template for the bundle ReadMe.txt generated by cmake_daily.sh
-rwxr-xr-x@ 1 rmills  staff   2.3K 16 Dec 19:41 spread*                   <---- Used to update the 'Categorized' builds

Detecting if svn has updated a branch
-------------------------------------

a=$(svn info . | grep ^Revision | cut -d: -f 2)
b=$(svn update . | grep ^At | cut '-d ' -f 3|cut -d. -f 1)
if [ $a -eq $b ]; then
	echo 'no build needed svn = ' $a
else
	echo 'updated from svn:' $a 'to svn:' $b
fi

Buildserver Configuration
-------------------------

The buildserver jobs configurations are located here:

523 rmills@rmillsmbp:/mmHD/Users/Shared/Jenkins/Home/jobs $ dir
drwxr-xr-x  1 rmills  staff   296B 16 Dec 10:22 Exiv2-trunk/
drwxr-xr-x  1 rmills  staff   296B 16 Dec 10:22 test-cmake-daily/
drwxr-xr-x  1 rmills  staff   264B 16 Dec 02:47 trunk-cmake-daily/
drwxrwxrwx  1 rmills  staff   296B 13 Dec 23:07 Exiv2-videow-refactoring/
drwxr-xr-x  1 rmills  staff   296B 12 May  2015 Exiv2-jenkins/
drwxr-xr-x  1 rmills  staff   330B  2 May  2015 Exiv2-video-write/

540 rmills@rmillsmbp:/mmHD/Users/Shared/Jenkins/Home/jobs/Exiv2-trunk $ dir
-rw-r--r--  1 rmills  staff   8.1K 16 Dec 10:22 config.xml                <---- Build magic
drwxr-xr-x  1 rmills  staff   264B 29 Dec  2014 configurations/           <---- History of config/build
drwxr-xr-x  1 rmills  staff   1.2K 15 Dec 22:38 builds/                   <---- Logs
-rw-r--r--  1 rmills  staff    78B 16 Dec 10:26 scm-polling.log           <---- name says it all
-rw-r--r--  1 rmills  staff     5B 15 Dec 22:26 nextBuildNumber
-rw-r--r--  1 rmills  staff    46B 15 Dec 22:26 svnexternals.txt          <---- don't know (nothing important)
lrwxr-xr-x  1 rmills  staff    22B 15 Dec 22:26 lastStable@ -> builds/lastStableBuild
lrwxr-xr-x  1 rmills  staff    26B 15 Dec 22:26 lastSuccessful@ -> builds/lastSuccessfulBuild

I've added Exiv2-trunk/config.xml to the repos.
I'll add something to jenkins_build.sh to keep the copy in the repos up to date.

Buildserver Scripts
-------------------

	Stable scripts (unlikely to change)
		Job:Exiv2-trunk (and selected branches)
		Script:
			./contrib/buildserver/jenkins_build.sh

		Trigger: when svn trunk changes

		Caution:
			older branches have jenkins_build.sh and jenkins_build.bat
			in the <exiv2dir> root directory

		Comment:
			This script builds when code is submitted.
			All platforms are automatically triggered by Jenkins and run jenkins_build.sh
			(Windows runs the 64 bit cygwin SSHD which uses 64 bit bash)
			Linux, MacOSX, Cygwin execute their builds directly from jenkins_build.sh
			MSVC builds are executed by jenkins_build.bat which is called via cmd.exe from jenkins_build.sh
			MinGW builds are executed by jenkins_build.sh calling jenkins_build.sh via MinGW{32|64}bash.exe

	Work in progress script:
		Job: trunk-cmake-daily:
		Trigger: 2am every day
		Script:
			cd ~/gnu/exiv2/buildserver
			a=$(/usr/local/bin/svn info .   | grep ^Revision | cut '-d:' -f 2                | tr -d ' ')
			b=$(/usr/local/bin/svn update . | grep ^At       | cut '-d ' -f 3 | cut -d. -f 1 | tr -d ' ')
			if [ "$a" == "$b" ]; then
			  echo ==================================
			  echo 'no build needed svn = ' $a
			  echo ==================================
			else
			  b=$(/usr/local/bin/svn info .   | grep ^Revision | cut '-d:' -f 2                | tr -d ' ')
			  echo ==================================
			  echo 'updated from svn:' $a 'to svn:' $b
			  echo ==================================
			  ssh rmills@rmillsmm                         'cd ~/gnu/exiv2/buildserver ; /usr/local/bin/svn update . ; rm -rf build ; contrib/buildserver/cmake_daily.sh'
			  ssh rmills@rmillsmm-kubuntu                 'cd ~/gnu/exiv2/buildserver ; /usr/local/bin/svn update . ; rm -rf build ; contrib/buildserver/cmake_daily.sh'
			  ssh rmills@rmillsmm-w7                      'cd ~/gnu/exiv2/buildserver ; /usr/local/bin/svn update . ; rm -rf build ; contrib/buildserver/cmake_daily.sh'
			  ssh rmills@rmillsmm-w7 'export PLATFORM=msvc;cd ~/gnu/exiv2/buildserver ; /usr/local/bin/svn update . ; rm -rf build ; contrib/buildserver/cmake_daily.sh'
			  ##
			  # test the delivery
			  date=$(date '+%Y-%m-%d+%H-%M-%S')
			  svn=$(/usr/local/bin/svn info . | grep ^Revision | cut -d: -f 2 | tr -d ' ')
			  (
				ssh rmills@rmillsmm                         '~/gnu/exiv2/buildserver/contrib/buildserver/test_daily.sh'
				ssh rmills@rmillsmm-kubuntu                 '~/gnu/exiv2/buildserver/contrib/buildserver/test_daily.sh'
				ssh rmills@rmillsmm-w7                      '~/gnu/exiv2/buildserver/contrib/buildserver/test_daily.sh'
				ssh rmills@rmillsmm-w7 'export PLATFORM=msvc;~/gnu/exiv2/buildserver/contrib/buildserver/test_daily.sh'
			  ) | tee "/mmHD/Users/Shared/Jenkins/Home/userContent/builds/Daily/test-svn-${svn}-date-${date}.txt"
			  ##
			  # categorize the builds
			  ssh rmills@rmillsmm         '~/gnu/exiv2/buildserver/contrib/buildserver/categorize.sh /mmHD/Users/Shared/Jenkins/Home/userContent/builds'
			fi


		Comment:
			This script builds once a day and ultimately publishes the build.

			For the moment, it's intended to run on trunk.  However the design
			can be "tweaked" to publish to builds/$JOBNAME/Daily etc

			I don't want to run this on code submission as the build takes about 1.5 hours
			to build all 12 Visual Studio builds (2005/2008/2010/2012/2013/2015) 32/64

			Currently MinGW has not been implemented.

			There are three quite different parts of the script:
			1) Use CMake to build on each platform using cmake_daily.sh
			2) Validate each platform using test_daily.sh
			3) Categorize the build (create links for Platform/SVN/Date/Latest)

			At the moment, the "pruning" of the builds is performed by cmake_daily.sh

Theme.css
---------

body {
     margin-top : 0px;  /* the height of your site banner */
     background	: lightskyblue;
     color		: black;
}


#header::after {
  content		: "";
  background	: url("Exiv2Logo50Background.png");;
  opacity		: 0.05;
  top			: 0;
  left			: 0;
  bottom		: 0;
  right			: 0;
  position		: absolute;
  z-index		: -1;
}

Notes concerning MinGW
----------------------

#########################################
##          #!/bin/bash
##          # mingw32.sh
##          # invoke 32bit MinGW bash
##          #
##          export "PATH=c:\\MinGW\\bin;c:\\MinGW\\msys\\1.0\\bin;C:\\MinGW\\msys\\1.0\\local\\bin;"
##          /cygdrive/c/MinGW/msys/1.0/bin/bash.exe $*
##
##          # That's all Folks
##          ##
#########################################

#########################################
##          : mingw32.bat
##          : invoke MinGW bash
##          :
##          setlocal
##          set "PATH=c:\MinGW\bin;c:\MinGW\msys\1.0\bin;C:\MinGW\msys\1.0\local\bin;"
##          set "PS1=\! ${PWD}> "
##          c:\MinGW\msys\1.0\bin\bash.exe %*%
##
##          : That's all Folks
#########################################

#########################################
##          see http://clanmills.com/exiv2/mingw.shtml about 64bit build
##          Install a fresh (32 bit) mingw/msys into c:\MinGW64
##          install the 64 bit compiler from: http://tdm-gcc.tdragon.net
##          I used the "on-demand" installer and "Create" put the tools in c:\TDM-GCC-64. The main change is to add the 64 bit compilers to the path BEFORE the 32 bit compilers.
##          set PATH=c:\TDM-GCC-64\bin;c:\MinGW\bin;c:\MinGW\msys\1.0\bin;C:\MinGW\msys\1.0\local\bin;
##
##          keep MinGW64 for 64 bit builds and /usr/lib has 64bit libraries
##          keep MinGW   for 32 bit builds and /usr/lib has 32bit libraries
##
##          install msys-coreutils, binutils, autotools
##
##          For pkg-config see http://clanmills.com/exiv2/mingw.shtml
#########################################

#########################################
##          zlib and expat
##          mkdir -p ~/gnu/zlib ~/gnu/expat
##          get the tar.gz files and tar zxf them
##          build (see http://clanmills.com/exiv2/mingw.shtml about zlib)
##          DO THIS IN BOTH c:\MinGW and c:\MinGW64
#########################################

#########################################
##          The keith bug
##          rm -rf /c/MinGW/lib/libintl.la
#########################################

#########################################
##          to build dlfcn-win32
##          git clone https://github.com/dlfcn-win32/dlfcn-win32
##          cd dlfcn-win32 ; ./configure --prefix=/usr --enable-shared ; make ; make install
#########################################



Robin Mills
robin@clanmills.com
2015-12-17
