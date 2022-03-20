# ranger-trash
A very small tweak of the delete build-in function to get a better chance to recover a file deleted by mistake

This very small tweak for [ranger](https://ranger.github.io) file manager that modify the delete build-in function to move deleted files to the linux /tmp di>

## Installation

Just copy the actions.py file and overwrite the native one in /usr/lib/python3.10/site-packages/ranger/core
It should take effect immediatly. In case it is not, restart the computer.

Here is the mmodified delete function (only 5 lines of difference...) which calls now shutil.move instead of shutil.rmtree :

```bash
    def delete(self, files=None):
        # XXX: warn when deleting mount points/unseen marked files?
        # COMPAT: old command.py use fm.delete() without arguments
        if files is None:
            files = (fobj.path for fobj in self.thistab.get_selection())
        self.notify("Delete {}!".format(", ".join(files)))
        # files = [os.path.abspath(path) for path in files]
        for path in files:
            # Untag the deleted files.
            for tag in self.fm.tags.tags:
                if str(tag).startswith(path):
                    self.fm.tags.remove(tag)
        self.copy_buffer = set(fobj for fobj in self.copy_buffer if fobj.path not in files)
        for path in files:
            _path=os.path.abspath(path)
            if not os.path.islink(_path):
                try:
                    file_size = os.path.getsize(_path)
                    if file_size < 1048576:
                        tmp='/tmp/'
                        new_name = next_available_filename(path,tmp)
                        new_dir = tmp + new_name
                        try:
                            shutil.move(_path,new_dir)
                        except OSError as err:
                            self.notify(err)
                    else:
                        if isdir(_path):
                            try:
                                shutil.rmtree(_path)
                            except OSError as err:
                                self.notify(err)
                        else:
                            try:
                                os.remove(_path)
                            except OSError as err:
                                self.notify(err)
                except OSError as err:
                    self.notify(err)
            else:
                try:
                    os.remove(_path)
                except OSError as err:
                    self.notify(err)
        self.thistab.ensure_correct_pointer()
```
