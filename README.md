![stem](https://s3-ap-southeast-2.amazonaws.com/stem-formatics/stem_formatics.PNG)

The controller code responsible for rendering the plotting functionality for a 3D In vitro model of a functional epidermal permeability barrier from human embryonic stem cells and induced pluripotent stem cells: [view](https://www.stemformatics.org/expressions/result?graphType=box&gene=ENSG00000115415&db_id=59&datasetID=6566#)


```
  def feature_result(self): 
        self._get_inputs_for_graph() 
        self._check_dataset_status()
        c.dataset_status = self._temp.dataset_status 
        c.chip_type = Stemformatics_Dataset.getChipType(db,c.ds_id)
        c.ref_type = self._temp.ref_type = self._temp.feature_type
        c.symbol = c.ref_id = self._temp.ref_id = self._temp.feature_id
        show_limited = True
        if self._temp.ref_type == 'miRNA':
            data_type = self._temp.ref_type
            c.datasets = Stemformatics_Dataset.get_all_datasets_of_a_data_type(c.uid,data_type,c.db_id)
        else:
            c.datasets = Stemformatics_Dataset.getChooseDatasetDetails(db,c.uid,show_limited,c.db_id) 
        # if for some reason such as db_id None or incorrect or ref_type incorrect, no datasets are fetched, user will be redirected to error message
        if c.datasets == {}:
            c.title = "Invalid miRNA Search"
            c.message = "You have not entered correct parameters. Please check the url if it has been entered manually"
            return render ('workbench/error_message.mako')

        c.probe_name = Stemformatics_Dataset.get_probe_name(c.ds_id)0

        # self._temp.this_view = self._setup_graphs(self._temp)
        # self._set_outputs_for_graph()
        audit_dict = {'ref_type':'feature_id','ref_id':self._temp.feature_id,'uid':c.uid,'url':url,'request':request}
        result = Stemformatics_Audit.add_audit_log(audit_dict)
        audit_dict = {'ref_type':'ds_id','ref_id':self._temp.ds_id,'uid':c.uid,'url':url,'request':request}
        result = Stemformatics_Audit.add_audit_log(audit_dict)

        return render('/expressions/result.mako')

```

## What’s good about this code: 

-	It will behave in a predictable manner. The function will grab the required inputs/parameters, check the validity of the dataset and store values locally in the context of itself.  It checks for a certain value of its reference type and if what its looking for is returned, it sets that as the ‘data type’. This is used as a filter to search using certain criteria (uid, data_type, dbid). 
-	It handles errors correctly and utilises the inbuilt mechanism included with pyramid (don’t re-invent the wheel)


## What’s not so good about this code:
-	At first glance, this code is quite difficult to read. The line spacing is poor and there is very little in-code commenting. Python is great because its so readable.  (See PEP8)
-	From a front end and performance point of view, I think the use of the mako templating framework could be reviewed. There seems to be a large amount of logic being processed on the front-end. Ideally, you want to load and filter/manipulate all your results in your view/controller, and have JavaScript responsible for representing the data. 

## How the code might be written differently if we want to take advantage of more modern javascript frameworks such as vuejs, so that we can make changes on the cllient side (html page) more easily.

-	From previous Django projects I have worked on, I have used the Jinja2 templating language for rendering data on the front-end. Jijna2 can be integrated with the pyramid framework, as can Chameleon and the default Mako. Jinja has some great advantages in terms of being distinctly readable when passed to your HTML page. Mako allows you to write application logic inside your html code, but it can be difficult to read and impacts performance. 
-	If the goal is to increase performance and improve or future proof the client side of the application, my recommendation moving forward would be to explore using jinga or similar. Below is an example of how to pass json data to your front end JS/Vue code. 


```
class ChartData(APIView):
    
    """
    Returns data from the API endpoint. Passes values into variables to be used in template via AJAX. 
    """
    
    authentication_classes = []
    permission_classes = []
    
    def get(self, request, format=None):
        
        users = Users.objects.all().count()
        comments = Comments.objects.all().count()
        features = Features.objects.all().count()
        bugs = Bugs.objects.all().count()
        
        labels = ["Users", "Comments", "Features", "Bugs"]
        default_items = [users, comments, features, bugs]
        
        data = {
                "labels": labels,
                "default_items": default_items,
            }
        return Response(data)
```

The data is manipulated within the function then passed to the 'data' variable in JSON format. 


```

    <div class="container">
        <div class="jumbotron" style="background-color:#ccd3e0;">
            <canvas id="myChart" width="400" height="400"></canvas>
        </div>
    </div>

    <script>
        
                var endpoint = '/ChartData/'
                var defaultData = []
                var labels = [];
                
                $.ajax({
                    method:"GET",
                    url: endpoint,
                    success: function(data){
                        labels = data.labels
                        defaultData = data.default_items
                        var ctx = document.getElementById("myChart").getContext('2d');
                        var myChart = new Chart(ctx, {
                            type: 'bar',
                            data: {
                                labels: labels,
                                datasets: [{
                                    label: 'Usage Data',
                                    data: defaultData,
                                    backgroundColor: [
                                        'rgba(255, 99, 132, 0.2)',
                                        'rgba(54, 162, 235, 0.2)',
                                        'rgba(255, 206, 86, 0.2)',
                                        'rgba(75, 192, 192, 0.2)',
                                        'rgba(153, 102, 255, 0.2)',
                                        'rgba(255, 159, 64, 0.2)'
                                    ],
                                    borderColor: [
                                        'rgba(255,99,132,1)',
                                        'rgba(54, 162, 235, 1)',
                                        'rgba(255, 206, 86, 1)',
                                        'rgba(75, 192, 192, 1)',
                                        'rgba(153, 102, 255, 1)',
                                        'rgba(255, 159, 64, 1)'
                                    ],
                                    borderWidth: 1
                                }]
                            }
                        })
                    
                    },
                    error: function(error_data){
                        console.log("error")
                        console.log(error_data)
                    }
                })
    </script>

```
