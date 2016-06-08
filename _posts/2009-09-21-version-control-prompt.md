---

title: Version Control Prompt
tags:
- tips
---
I find it convenient to include current SCM data before my regular Bash prompt
(reduces the chance of "accidents"). Perhaps someone else will find it useful
too.

    
    function prompt-pre-exec() {
        scm=""
        repo_root=$(hg root 2>/dev/null)
        if [ -e CVS ]; then
            scm=":: cvs ::"
    
        elif [ -e .svn ]; then
            scm=":: svn : ${prompt_hl}r$(svn info | grep Revision | sed s/.*:\ //)${prompt_n} ($(svn info | grep Date | sed s/.*\(\//)"
    
        elif [ -e .gitignore ]; then
            repo_branch=`git branch --no-color 2> /dev/null | sed -e /^[^*]/d -e s/* \(.*\)/\1#/`
            scm=`git show --pretty="format: : git : ${prompt_hl}${repo_branch}%h${prompt_n} : %an, %cr\n:: %s\n" | head -n 2`
    
        elif [ x != "x$repo_root" ]; then
            repo_cs=$(hg id -i)
            scm=`hg log --template " : hg : ${repo_root##*/} : ${prompt_hl}${repo_cs}${prompt_n} {tags} : {author|user}, {date|age} ago\n:: {desc|firstline|strip}\n" -r ${repo_cs%%+}`
        fi
    
        if [ "x$scm" != x ]; then
            # Trailing \n characters dont seem to expanded 
            scm="$scm
    "
        fi
        export scm    
    }
    
    if [ x"$-" = "xhimBH" ]; then
      # Execute the following function before displaying the prompt
      export PROMPT_COMMAND=prompt-pre-exec
    
      # Use \[ and \] to exclude the color code from the line wrapping calculations 
      export PS1=${scm}[\@] \u@\h \[${prompt_hl}\]\w #\[${prompt_n}\] 
    fi
    

Then to add color, simply define _prompt_hl_ and _prompt_n_. I use

    
    export prompt_n="^[^E[00m^]"      # Default color
    export prompt_hl="^[^E[01;32m^]"  # Highlight codes
    

To enter _^[_ in emacs, type Ctrl-q then Ctrl-[. Likewise _^E_ is Ctrl-q
Ctrl-e.

