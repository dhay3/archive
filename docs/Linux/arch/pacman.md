# pacman

参考：

https://wiki.manjaro.org/index.php/Pacman_Overview

https://wiki.archlinux.org/title/Pacman_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

package manager utility(pacman)，manjaro和arch都使用它作为包管理器。

syntax：`pacman <operations> [options] [target]`

target通常package name或是URI

> 注意manjaro只能使用official Manajaro repo，但是也可以使用ARU，flatpaks，snaps或appImages

## operations

### -D | --database

==查看和修改pacman的db==

```bash
cpl in ~ λ sudo pacman -Dv
Root      : /
Conf File : /etc/pacman.conf
DB Path   : /var/lib/pacman/
Cache Dirs: /var/cache/pacman/pkg/  
Hook Dirs : /usr/share/libalpm/hooks/  /etc/pacman.d/hooks/  
Lock File : /var/lib/pacman/db.lck
Log File  : /var/log/pacman.log
GPG Dir   : /etc/pacman.d/gnupg/
Targets   : 无
```

### -Q | --query

查询已经安装的pkg，如果没有指定pkg，默认查询所有安装的pkg

- -i | -q

  查看pkg的详细信息，`-q` show less info for query and search 

- -l

  查看pkg安装的所有内容(ps. 查看文件路径)

- -m | -n

  `-m`查看不是通过sync dbs安装的pkg(ps. not offical repo installed pkg)，`-n`查看通过sync dbs安装的pkg

- -s

  从pkg name和description中查找，通常和`-q`一起使用

- -u

  list packages can be upgraded

- -t

  列出不被需要的pkg

- -d

  列出所有被作为denpends的pkg

### -S | --sync

sync form remote server 。和apt一样如果一个package存在于多个pkg中，需要指明repo，例如`pacman -S <reponame>/<pkg>`。`pacman -S <pkg>`==同时做upgrade和install==

- -i 

  查看pkg的详细信息

- -c

  删除pacman保存的cache

- -s

  重remote server上查看指定pkg

  ```
  cpl in /etc/pacman.d λ pacman -Ssq firefox
  ```

- -u

  升级所有到达生命周期的pkg

- -y

  更新pkg db，通常和`-u`一起使用(==更新所有的pkg==)

  ```
  pacman -Syu
  ```

  使用`-yy`表示强制更新
  
  ```
  pacman -Syy
  ```

### -R | --remove

删除pkg，默认不会删除配置文件(和`apt-get remove`)类似，所有的配置文件以`.pacsave`结尾。使用`--nosave`等价于`apt-get purne`。

- -c

  删除pkg时，删除depends(==所有的依赖==)

- -n | --nosave

  ==删除pkg时同时删除配置文件==

- -s

  删除pkg中不被==其他包==需要的depends

- -u 

  删除pkg(==本包==)不再需要的depends

### -U | --upgrade

安装一个本地包，通常和`makepkg`一起使用安装AUR BUILDPKG

```
pacman -U /path/to/package/package_name-version.pkg.tar.xz
```

## remove orphans

pacman没有apt的autoremove的功能，但是可以通过`pacman -Rs $(pacman -Qdtq)`来达到autoremove，也可以使用如下的脚本：

```shell
LOOPFLAG=0
PACMAN=$(which pacman 2> /dev/null)
SUDO=$(which sudo 2> /dev/null)
 
case "$1" in                      
  -l)
  echo -e "
  \r** UNNEEDED DEPENDENCIES **
  \r-> checking dependencies...
  "
  $PACMAN -Qdtq
  if [ "$?" = 1 ]; then
    echo -e "-> Your system doesn't have unneeded dependencies. \n"
  fi 
  ;;
  -r)
  while [ "$LOOPFLAG" = 0 ]
  do
    echo -e "
    \r** UNNEEDED DEPENDENCIES **
    \r-> checking dependencies...
    "
    $PACMAN -Qdtq
    if [ "$?" = 0 ]; then
      echo ""
      echo -n "Remove these packages with pacman? [Y/n] "
      read OPTION 
      if [ "$OPTION" = "y" ] || [ "$OPTION" = "" ]; then
        echo -n "-> "
        if [ -f $SUDO ]; then
          $SUDO $PACMAN -Rn $($PACMAN -Qdtq)
          if [ "$?" != 0 ]; then
            echo -e "-> Dependencies skipped... next dependencies... \n"
          else
            echo -e "-> Unneeded dependencie(s) sucessfully removed. \n"
          fi
        else
          $PACMAN -Rn $($PACMAN -Qdtq)
          echo -e "-> Unneeded dependencie(s) sucessfully removed. \n"
        fi
      elif [ "$OPTION" = "n" ]; then
        exit 0
      fi  
    else
      LOOPFLAG=1
      echo -e "-> Your system doesn't have unneeded dependencies. \n"
    fi
  done
  ;;
  -ra)
  $PACMAN -Qdtq > /dev/null
  if [ "$?" = 1 ]; then
    echo -e "
    \r** UNNEEDED DEPENDENCIES **
    \r-> checking dependencies...
    "
    echo -e "-> Your system doesn't have unneeded dependencies. \n"    
  else  
    echo -e "\n** UNNEEDED DEPENDENCIES - RECURSIVE **"
    echo -n "-> "
    if [ -f $SUDO ]; then
       $SUDO $PACMAN -Rsn $($PACMAN -Qdtq)
    else
       $PACMAN -Rsn $($PACMAN -Qdtq)
    fi
  fi
  ;;
  *)
    echo "Usage: $0 {-l <list> | -r <remove> | -ra <remove all - recursive>}"
esac
exit 0
```



