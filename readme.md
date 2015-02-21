
## Launch Master

```
$ salt-master --config-dir=. --user=$USER
```

## Launch Minions

```
$ salt-minion --config-dir=minion1 --user=$USER
$ salt-minion --config-dir=minion2 --user=$USER
$ salt-minion --config-dir=minion3 --user=$USER
```

## List files

```yaml
$ salt --config-dir=. --user=$USER 'minion1' cp.list_master
minion1:
    [snip]
```

## gitfs file

```yaml
$ salt --config-dir=. --user=$USER 'minion1' cp.get_file_str salt://salt/utils/__init__.py | head
minion1:
    # -*- coding: utf-8 -*-
    '''
    Some of the utils used by salt
    '''

    # Import python libs
    from __future__ import absolute_import, print_function
    import contextlib
    import copy
```

## Hammer time

So this is surely crazy, but the ocassional error on the salt-minion is the same as I've seen in production with 15 minions and running a `state.highstate`.

```bash
$ seq 1 100 | parallel --will-cite --halt-on-error -j3 "salt --config-dir=. --user=$USER '*' cp.get_file_str salt://salt/utils/__init__.py; true || " | grep "until if has a working string that does not stack trace" | wc -l
```

## Errors

### From salt-master

```yaml
[ERROR   ] Exception [Errno 2] No such file or directory: '/Users/jason/repos/jasonrm/salt-gitfs-locks/var/cache/salt/master/gitfs/220cae4dc85532ff6c9a6e25428d827a/update.lk' occurred in file server update
[ERROR   ] Error in function _file_hash:
Traceback (most recent call last):
  File "/usr/local/lib/python2.7/site-packages/salt/master.py", line 1336, in run_func
    ret = getattr(self, func)(load)
  File "/usr/local/lib/python2.7/site-packages/salt/fileserver/__init__.py", line 423, in file_hash
    fnd = self.find_file(load['path'], load['saltenv'])
  File "/usr/local/lib/python2.7/site-packages/salt/fileserver/__init__.py", line 377, in find_file
    fnd = self.servers[fstr](path, saltenv, **kwargs)
  File "/usr/local/lib/python2.7/site-packages/salt/fileserver/gitfs.py", line 1353, in find_file
    return fnd
  File "/usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Versions/2.7/lib/python2.7/contextlib.py", line 24, in __exit__
    self.gen.next()
  File "/usr/local/lib/python2.7/site-packages/salt/fileserver/gitfs.py", line 916, in _aquire_update_lock_for_repo
    yield
  File "/usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Versions/2.7/lib/python2.7/contextlib.py", line 24, in __exit__
    self.gen.next()
  File "/usr/local/lib/python2.7/site-packages/salt/fileserver/gitfs.py", line 932, in wait_for_write_lock
    os.remove(filename)
OSError: [Errno 2] No such file or directory: '/Users/jason/repos/jasonrm/salt-gitfs-locks/var/cache/salt/master/gitfs/8ac80fc2091a1e0ce942c76abdf214a7/update.lk'
```

### From salt-minion

```yaml
[ERROR   ] Data is
[WARNING ] TypeError encountered executing cp.get_file_str: string indices must be integers, not str. See debug log for more info.
```
