# Django Live Project

### Introduction
Working on a two week sprint has really shown me the workflow, as well as the proper techniques professional developers use to create Web Applications using Python (my personal favorite language, don’t tell the other languages) and the Django framework. As I moved forward through the sprint, the short two weeks has really shown me how to work as a team and help out teammates, as well as being able to ask questions with confidence and understand the answers you receive. Doing Full Stack development I really took advantage of this project to understand the connection between [front-end](#Front-End-Stories) and [back-end](#Back-End-Stories) development, as well as best practices between the two.

Below are some examples of the type of work and a few problems and how I choose to resolve them. This does not include the whole project but instead shows snippets of my code, and include descriptions of bugs I solved along the way.

### Back-End Stories
[Models](#Models)\
[Forms](#Forms)\
[API Function](#API-Function)\
[Total Food](#Total-Food)

#### Models
Although we were required to have one table in our database for our projects, I choose to utilize two tables to create a better and more personal experience, logging both the daily exercises for the user along with the daily nutrition(e.g. fats, carbs, or protein). 

```
date = datetime.datetime.now()


WORKOUT_TYPE = {
    ('Cardio', 'Cardio'),
    ('Strength Training', 'Strength Training'),
    ('HITT', 'HITT'),
    ('Yoga', 'Yoga'),
    ('Sports', 'Sports'),
}

# Daily Table is to store food log for calculating totals and recall
class Daily(models.Model):
    date = models.DateField(auto_now_add=True)
    food_name = models.CharField(max_length=60, default="")
    food_fats = models.FloatField(blank=True, null=True)
    food_carbs = models.FloatField(blank=True, null=True)
    food_protein = models.FloatField(blank=True, null=True)
    food_calories = models.FloatField(blank=True, null=True)

    Foods = models.Manager()

    def __str__(self):
        return self.food_name


# Activity Table is to store your activities and times for the day. Including (optional) Weight
class Activity(models.Model):
    date = models.DateField(default=date.strftime("%m/%d/%Y"))
    workout_type = models.CharField(max_length=60, choices=WORKOUT_TYPE)
    length = models.DurationField(max_length=5, default='0:00:00')
    weight = models.DecimalField(max_digits=5, decimal_places=2, blank=True, null=True)

    Activities = models.Manager()

    def __str__(self):
        return self.workout_type
```
<hr>

#### Forms
This also allowed me to take advantage of Djangos ability to create forms and save the data received from the API for recall of the daily nutrition for the user.

```
class ActivityForm(ModelForm):
    class Meta:
        model = Activity
        fields = '__all__'


class FoodForm(ModelForm):
    class Meta:
        model = Daily
        exclude = ['date', 'food_name']
```
<hr>

#### API Function
For my API I chose [nutritionix.com](https://www.nutritionix.com/), which was a great experience working with an API that had slightly more strict requirements for the type of information sent in order to get a response.

```
def get_nutrition(food):
    connection = http.client.HTTPSConnection('trackapi.nutritionix.com')
    print(food)
    headers = {
        'accept': 'application/json',
        'x-app-id': ID_TOKEN,
        'x-app-key': SECRET_TOKEN,
        'x-remote-user-id': REMOTE_TOKEN,
        'Content-Type': 'application/json',
    }
    body = {
        'query': food,
    }
    body_json = json.dumps(body)
    print(headers, body) # For debugging and to see terminal response
    connection.request('POST', '/v2/natural/nutrients', body_json, headers)
    response = json.loads(connection.getresponse().read().decode())
    print(response)
    return response
```
<hr>

#### Total Food
I also had to include a function to take all the food logged for that day only, and add up the nutrients and return the information to my main API function.

```
def get_food():
    food = Daily.Foods.all().filter(date=datetime.date.today())
    fats_total = 0
    carbs_total = 0
    protein_total = 0
    calories_total = 0
    for i in food:
        fats_total += i.food_fats
        carbs_total += i.food_carbs
        protein_total += i.food_protein
        calories_total += i.food_calories
    data = {
        'fats_total': fats_total,
        'carbs_total': carbs_total,
        'protein_total': protein_total,
        'calories_total': calories_total,
    }
    
    return data
```
<hr>

### Front-End Stories
Front end design was pretty simple integrating my designs in the main CSS file without effecting other pages, but still using the same basic template. The trick came in passing information into manually rendered forms and then saving that information to the database for later recall. Below is my HTML template for displaying an autofill form with the information from the API and then saving that data once an “Add Food” button is clicked. Also the “Add Food” button is only displayed once a food has been searched.

```
<section>
    <div class="flex-container">
        <form method="POST" class="nutritionCalcForm">
            {% csrf_token %}
            <table class="inputTable" id="input">
                {{form.as_p}}
            </table>
            {{ form.non_field_errors }}
            <button class="primary-bright-button" id="submitBtn" type="submit" value="submit">Search food</button>
        </form>

        <div class="outputTable">
            <form method="POST">
                {% csrf_token %}
                <table id="displayTable">
                    {% for item in search %}
                        <tr>
                            <th><label for="food_name">Food Name:</label></th>
                            <td><input type="text" name="food_name" id="food_name" value="{{item.food_name}}"></td>
                        </tr>
                        <tr>
                            <th><label>Grams:</label></th>
                            <td><input value="{{item.serving_weight_grams}}"></td>
                        </tr>
                        <tr>
                            <th><label for="food_fats">Total Fat:</label></th>
                            <td><input type="number" name="food_fats" id="food_fats" value="{{item.nf_total_fat}}"></td>
                        </tr>
                        <tr>
                            <th><label for="food_carbs">Total Carbohydrates:&nbsp&nbsp&nbsp&nbsp</label></th>
                            <td><input type="number" name="food_carbs" id="food_carbs" value="{{item.nf_total_carbohydrate}}"></td>
                        </tr>
                        <tr>
                            <th><label for="food_protein">Total Protein:</label></th>
                            <td><input type="number" name="food_protein" id="food_protein" value="{{item.nf_protein}}"></td>
                        </tr>
                        <tr>
                            <th><label for="food_calories">Total Calories:</label></th>
                            <td><input type="number" name="food_calories" id="food_calories" value="{{item.nf_calories}}"></td>
                        </tr>
                        {% if search != None %}
                            <button class="primary-bright-button" id="addBtn" type="submit">Add Food</button>
                        {% endif %}
                    {% endfor %}
                </table>
            </form>
            <form name="totalTable" id="totalTable">
                {% csrf_token %}
                <table>
                    {% for item in total %}
                        <tr>
                            <th><label>Todays Nutrition Totals:</label></th>
                        </tr>
                        <tr>
                            <th><label>Total Fat:</label></th>
                            <td>{{item.fats_total}}</td>
                        </tr>
                        <tr>
                            <th><label>Total Carbohydrates:&nbsp&nbsp&nbsp&nbsp</label></th>
                            <td>{{item.carbs_total}}</td>
                        </tr>
                        <tr>
                            <th><label>Total Protein:</label></th>
                            <td>{{item.protein_total}}</td>
                        </tr>
                        <tr>
                            <th><label>Total Calories:</label></th>
                            <td>{{item.calories_total}}</td>
                        </tr>
                    {% endfor %}
                </table>
            </form>
        </div>
    </div>
</section>
```
<hr>

### Bugs/Learned
As I mentioned above, my API would only send a response if the body of my request was formatted correctly. Although I was sending the correct information to the API, the body of my request needed to be formatted as a JSON before the request was sent. This was simply solved with these few lines.

```
body = {
        'query': food,
    }
    body_json = json.dumps(body)
```
<hr>

### Conclusion/Gained Skills
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Working as a team creating a functional and useful web application.\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Creating forms that can be saved, edited, and deleted from the database.\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Looking at different ways and implementing the most simple and efficient way to solve problems and bugs.\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Creating new branches and using clean version control.\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Organization of code and readability.
