```python

class UnstructuredMarkdownLoader(UnstructuredFileLoader):
    def _get_elements(self) -> List:
        from unstructured.__version__ import __version__ as __unstructured_version__
        from unstructured.partition.md import partition_md

        _unstructured_version = __unstructured_version__.split("-")[0]
        unstructured_version = tuple([int(x) for x in _unstructured_version.split(".")])

        return partition_md(filename=self.file_path, **self.unstructured_kwargs)

```