# KiCad
Here is a collection of notes about the peculiarities of KiCad I have found while learning to work with it.

## Missing 3D model of parts after STEP export
I had added some STEP models to some footprints in the library that were missing it and it all seemed to work fine until I exported the whole PCB project to a STEP file and all the models I had added were missing.
I had downloaded the official KiCad 3D models from the [GitHub repository <i class="fa fa-external-link"></i>](https://github.com/KiCad/kicad-packages3D) into a directory called `kicad-packages3D`.
At the same level I had made a directory called `custom` where I had put all the STEP files that were missing.
In the footprint editor for each component I added the correct STEP file located in the `custom` directory and it showed up in the KiCad 3D viewer but not after export.

The solution to this problem seems to be to put everything under the main `kicad-packages3D` directory instead.
If I make a directory called `kicad-packages3D/custom.3shapes` and add the new STEP files there instead and set the correct link in KiCad the STEP export of the PCB works!

!!! success "Working directory structure"
    ```shell
    |-kicad-packages3D
    |---custom.3shapes
    |-----part_1.step
    |-----part_2.step
    ```

!!! warning "Not working directory structure"
    ```shell
    |-kicad-packages3D
    |-custom
    |---part_1.step
    |---part_2.step
    ```