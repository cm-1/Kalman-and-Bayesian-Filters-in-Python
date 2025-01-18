Seems to work fine in Jupyter Lab so far, so using Jupyter Notebook might not
be required. Installation will then require ipywidgets, though.

In Jupyter Notebook (still neet to try Jupyter Lab), did not need to do much
extra to get `%matplotlib tk` to work. Not sure if my prior installation of
ipympl had any effect, since I recall things requiring more things to be
installed before to get the tk magic to work... maybe that was just with VS Code,
though, or maybe there's something in filterpy or another library that took
care of it.

On the subject of ipympl, it seems to work for some of the book's graphs but
quickly causes issues in later cells. At least, that's the case for the 01
chapter.
