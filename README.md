# NUISANCE containers

This repo builds a suite of containers designed to generate events for each of the four main neutrino generator packages used in the community, including all of their dependencies, plus NUISANCE. Each of the generator-specific subdirectories contain a single Dockerfile, plus any additional files needed for each generator build.
* `root_v6.30.08`: Builds PYTHIA6 and ROOT v6.30.08 on a debian-12:slim base container. The python3 environment is in `/opt/venv`. This acts as the base container for the other generator builds.
* `genie_v3.6.2`: This builds GENIE v3.6.2 and all of its dependencies, plus GENIERW v1.4.2, plus NUISANCE. Build objects etc are not removed, making this a large container suitable for development.
* `gibuu_v2025p5`: Builds GiBUU v2025p1 and dependencies. Build objects etc are not removed, making this a large container suitable for development.
* `nuwro_v25.11.01`: Builds NuWro v25.11.01 using a custom patch to remove a compilation error (a PR has been submitted). Build objects etc are not removed, making this a large container suitable for development.
* `neut_v5.9.2`: Builds NEUT v5.9.2 (including NEUT's reweighting) and CERNLIB as well as other dependencies. Build objects etc are not removed, making this a large container suitable for development.
* `tutorial`: This container contains the binaries and all runtime objects to run all of the generators, copied from the pre-build containers above, plus NUISANCE built on all of the above. Build objects are removed along with git history etc to make this relatively small, but not great as a development container.

Each can be built locally and pushed to a remote with, e.g.:
```
docker build -f Dockerfile -t wilkinsonnu/nuisancemc:<name> --progress=plain --network=host --push .
```
(remove `--push` to keep things local)

Github actions are defined in `.github/workflows` such that whenever one of the Dockerfiles is pushed to github, it rebuilds the container for both amd64 and arm64, and publishes the result at: https://hub.docker.com/r/wilkinsonnu/nuisancemc.

If anybody wishes to fork this repo, they should point all of the Dockerfiles at a dockerhub they have access to, modify the YAML files that control the workflows, and add the relevant dockerhub key under github actions.

