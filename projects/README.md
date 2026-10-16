# projects

One folder per product's video rig, made by `python3 skills/demo-video/scripts/new_project.py projects/<name>`: its `record.py` (the scenario), `edit.py` (the cut) and `captions.json`.

Everything in here except this README is git-ignored, because a rig names a product, its screens and its URLs. To keep your rigs under version control, commit them to a private fork, or force-add them (`git add -f`) knowing what they contain.

Raw takes don't go here: they live in `~/recordings/<name>/`, outside every repository.

A rig finds the kit through the absolute path `new_project.py` wrote on its `sys.path.insert` line. If you move the kit, update that line in the rig's `record.py` and `edit.py`.
