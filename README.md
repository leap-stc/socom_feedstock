# SOCOM Feedstock

## Setup
### Use this template
- Click on the button on the top left to use this repository as a template for your new feedstock
<img width="749" alt="image" src="https://github.com/leap-stc/proto_feedstock/assets/14314623/c786b2c7-adf1-4d4c-9811-0c7a1aa9228c">

>[!IMPORTANT]
> - Make the repo public
> - Make sure to create the repo under the `leap-stc` github organization, not your personal account!
> - Name your feedstock according to your data  `<your_data>_feedstock`.
>
>  If you made a mistake here it is not a huge problem. All these settings can be changed after you created the repo.

- Now you can locally check out the repository.

> [!NOTE]
> The instructions below are specific for testing recipes locally but downloading and producing data on GCS cloud buckets. If you are running the recipes locally you have to minimally modify some of the steps as noted below.

### Build and test your recipe locally on the LEAP-Pangeo Jupyterhub

- Edit the `feedstock/recipe.py` to build your pangeo-forge recipe. If you are new to pangeo-forge, [the docs](https://pangeo-forge.readthedocs.io/en/latest/composition/index.html#overview) are a great starting point
- Make sure to also edit the other files in the `/feedstock/` directory. More info on feedstock structure can be found [here](https://pangeo-forge.readthedocs.io/en/latest/deployment/feedstocks.html#meta-yaml)
- 🚨 You should not have to modify any of the files outside the `feedstock` folder (and this README)! If you run into a situation where you think changes are needed, please open an issue and tag @leap-stc/data-and-compute.

### Test your recipe locally
Before we run your recipe on LEAPs Dataflow runner you should test your recipe locally.

You can do that on the LEAP-Pangeo Jupyterhub or your own computer.

1. Set up an environment with mamba or conda:
```shell
mamba create -n runner0102 python=3.11 -y
conda activate runner0102
pip install pangeo-forge-runner==0.10.2 --no-cache-dir
```

2. You can now use [pangeo-forge-runner](https://github.com/pangeo-forge/pangeo-forge-runner) from the root directory of a checked out version of this repository in the shell

```shell
pangeo-forge-runner bake \
  --repo=./ \
  --Bake.recipe_id=<recipe_id>\
  -f configs/config_local_hub.py
```
>[!NOTE]
> Make sure to replace the `'recipe_id'` with the one defined in your `feedstock/meta.yaml` file.
>
>If you created multiple recipes you have to run a call like above for each one.

> To run this fully local (e.g. on your laptop) you have to replace `config_local_hub.py` with  `config_local.py`.
>
> ⚠️ This will save the cache and output to a subfolder of the location you are executing this from.. Make sure do delete them once you are done with testing.

3. Check the output! If something looks off edit your recipe.

>[!TIP]
>The above command will by default 'prune' the recipe, meaning it will only use two of the input files you provided to avoid creating too large output.
>Keep that in mind when you check the output for correctness.

Once you are happy with the output it is time to commit your work to git, push to github and get this recipe set up for ingestion using [Google Dataflow](https://cloud.google.com/dataflow?hl=en)

### Activate the linting CI and clean up your repo
[Pre-Commit](https://pre-commit.com) linting is already pre-configured in this repository. To run the checks locally simply do:
```shell
pip install pre-commit
pre-commit install
pre-commit run --all-files
```
Then create a new branch and add those fixes (and others that were not able to auto-fix). From now on pre-commit will run checks after every commit.

Alternatively (or additionally) you can use the  [pre-commit CI Github App](https://results.pre-commit.ci/) to run these checks as part of every PR.
To proceed with this step you will need assistance a memeber of the [LEAP Data and Computation Team](https://leap-stc.github.io/support.html#data-and-computation-team). Please open an issue on this repository and tag `@leap-stc/data-and-compute` and ask for this repository to be added to the pre-commit.ci app.

### Deploy your recipe to LEAPs Google Dataflow

>[!WARNING]
>To proceed with this step you will need to have certain repository secrets set up. For security reasons this should be done by a memeber of the [LEAP Data and Computation Team](https://leap-stc.github.io/support.html#data-and-computation-team). Please open an issue on this repository and tag `@leap-stc/data-and-compute` to get assistance.

- To deploy a recipe to Google Dataflow you have to trigger the "Deploy Recipes to Google Dataflow" with a single `recipe_id` as input.

- Once your recipe is run from a github workflow we assume that it is deployed to Google Dataflow and activate the final [copy stage](https://github.com/leap-stc/LEAP_template_feedstock/blob/55ee23ce0bc90f764d18bc34c58adccb5b38fc89/feedstock/recipe.py#L63). This happens automatically, but you have to make sure to edit the `'feedstock/catalog.yaml'` `url` entries for each `recipe_id`. This location will be the 'final' location of the data, and this is what gets passed to the the catalog in the next step!

>[!NOTE]
>By default the `'prune'` option is set to true. To build the final dataset you need to change that value [here](https://github.com/leap-stc/LEAP_template_feedstock/blob/55ee23ce0bc90f764d18bc34c58adccb5b38fc89/configs/config_dataflow.py#L7). **Particularly for large datasets make sure that you have finalized the entries in `'feedstock/catalog.yaml'`**, since the full build of the dataset can be slow and expensive - you want to avoid doing that again 😁

### Add your dataset to the LEAP-Pangeo Catalog
Now that your awesome dataset is available as an ARCO zarr store, you should make sure that everyone else at LEAP can check this dataset out easily.
