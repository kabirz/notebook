```python
import yaml
import os

configs = yaml.safe_load(open(os.path.expanduser('~/.dotfiles/ids/key.yaml')))
openai = configs['llm']['openai']
azure = configs['llm']['azure']

def get_model_infos(model_name):
    for _, ctx in openai.items():
        if model_name in ctx['models']:
            info = ctx['info']
            info['model'] = model_name
            return info
    raise ValueError('Can not find model for {}'.format(model_name))

def get_azure_infos(model_name, resoucre_name):
    for _resource_name, ctx in azure.items():
        if resoucre_name != _resource_name:
            continue
        if model_name in ctx['models']:
            info = ctx['info']
            info['deployment_name'] = model_name
            info['azure_endpoint'] =f'https://{resoucre_name}.openai.azure.com'
            return info
    raise ValueError('Can not find model for {}'.format(model_name))

models = []
for _, ctx in openai.items():
    models.extend(ctx['models'])
```
