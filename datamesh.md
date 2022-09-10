# Hands On: Creating Your Own Data Mesh
- `sudo apt-get install  git make jq docker`
- `git clone https://github.com/confluentinc/data-mesh-demo`
- `export PATH=$(pwd)/bin:$PATH`
- `confluent login --save`
- `cd data-mesh-demo`
- `make data-mesh`
- `make destroy`