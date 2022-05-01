# ranger-trash
A very small tweak of the delete build-in function of ranger to get a better chance to recover a file deleted by mistake.

This very small tweak for [ranger](https://ranger.github.io) file manager modifies the delete build-in function to move deleted files to the linux /tmp directory instead of deleting them straightaway. It is not a real "trash" as the files will be deleted at the end of the session as the tmp directory content, but it is enough to avoid any mistake forcing us to look for the inodes... To avoid moving large files, a limit of 100 Mb is put, which can be easily modified of course, editing the file at the line `if file_size < 104857600:...
Note than ranger have already a trash feature, which is a "real" trash. But I personally I don't like trash, quickly empeded with old files and using volume pointlessly.

## Installation

Just copy the actions.py file and overwrite the native one in /usr/lib/python3.10/site-packages/ranger/core
It should take effect immediatly. In case it is not, restart the computer.

Here is the modified delete function, which calls now shutil.move instead of shutil.rmtree :

```python
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
                    if file_size < 104857600:
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
