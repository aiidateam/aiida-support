---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.14.5
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

# Influence of setting `ibrav`

```python
from aiida import orm, load_profile
from aiida_quantumespresso.workflows.pw.base import PwBaseWorkChain
from pathlib import Path
from aiida.engine import run_get_node

from ase import io

load_profile()
```

```python
atoms = io.read(Path('scf-ibrav2.in'))
structure = orm.StructureData(ase=atoms)
code = orm.load_code('qe-7.2-pw@localhost')
```

## Run without `ibrav`

```python
builder = PwBaseWorkChain.get_builder_from_protocol(code=code, structure=structure)

results, node = run_get_node(builder)
```

```python
results['output_parameters'].get_dict()['wall_time']
```

```python
node.called[-1].base.repository.list_object_names()
print(node.called[-1].base.repository.get_object_content('aiida.in'))
```

## Run with `ibrav`

```python
builder = PwBaseWorkChain.get_builder_from_protocol(code=code, structure=structure)

builder.pw.parameters['SYSTEM']['ibrav'] = 2

# Note that this is not possible - Also see https://github.com/aiidateam/aiida-quantumespresso/issues/922 
# builder.pw.parameters['SYSTEM']['celldm(1)'] = 8.0374557182  

results, node = run_get_node(builder)
```

```python
results['output_parameters'].get_dict()['wall_time']
```

```python
print(node.called[-1].base.repository.get_object_content('aiida.in'))
```

## `parent_folder` creator

```python
# First run
builder = PwBaseWorkChain.get_builder_from_protocol(code=code, structure=structure)

results, node = run_get_node(builder)

# Restart
restart_builder = PwBaseWorkChain.get_builder_from_protocol(code=code, structure=structure)

parameters = restart_builder.pw.parameters.get_dict()
parameters['ELECTRONS']['startingpot'] = 'file'
restart_builder.pw.parameters = orm.Dict(parameters)
restart_builder.pw.parent_folder = node.outputs.remote_folder

restart_results, restart_node = run_get_node(restart_builder)

restart_node.inputs.pw.parent_folder.creator
```

You can also do the following, but only because the `Dict` node is not stored!

```python
parent_folder = node.outputs.remote_folder

restart_builder = PwBaseWorkChain.get_builder_from_protocol(code=code, structure=structure)

restart_builder.pw.parameters['ELECTRONS']['startingpot'] = 'file'
restart_builder.pw.parent_folder = parent_folder

test_restults, test_node = run_get_node(restart_builder)
```
