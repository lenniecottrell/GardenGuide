# GardenGuide

## Introduction

GardenGuide is a basic garden-tracking web app using Django, Bootstrap, and SQLite as part of a large collection of apps. All of the code was written over a two-week sprint for the Live Project course at The Tech Academy. The app is intended for users to enter information about the plants they have in their garden, including the common and scientific names, how the plant was started (e.g. from seed), what kind of light and water needs it has, etc. It also includes a page with data scraped from another gardening blog. This was a full stack project - I was repsonsible for the front-end design as well as the database structure and data processing. While I worked on it, I was part of a team that utilized Azure DevOps to collaborate and coordinate all of our apps into the larger project. 

I wasn't able to upload the live version of the project, since it's an ongoing event hosted by The Tech Academy, but here's an overview of my work.
 
## Front End

##### Item Index
I used Django's built-in template inheritance structure to build the basic layout of my app. I wanted to display information from two related tables onto a single page. `single_plant_growing` is an instantiated object from the child table, and `plantdetails` is my foreign key. To access data in the parent table (to get the common name, for example), I needed to reference the foreign key using dot notation. 

    {% block maincontent %}

            {% comment %}
                This page lists all of the data for for a single item.
                At the bottom there are buttons to update or delete information
            {% endcomment %}

        <b>{{ single_plant_growing.plantdetails.common_name | title }}</b>

        <table class="table">
            <tr>
                <td class="text-center text-white mt-5">Family:</td>
                <td class="text-white text-center mt-5">{{ single_plant_growing.plantdetails.family }}</td>
            </tr>
            <tr>
                <td class="text-center text-white mt-5">Scientific Name:</td>
                <td class="text-white text-center mt-5">{{ single_plant_growing.plantdetails.scientific_name }}</td>
            </tr>
            <tr>
                <td class="text-center text-white mt-5">Duration:</td>
                <td class="text-white text-center mt-5">{{ single_plant_growing.plantdetails.duration }}</td>
            </tr>
            <tr>
                <td class="text-center text-white mt-5">Light:</td>
                <td class="text-white text-center mt-5">{{ single_plant_growing.light_type }}</td>
            </tr>
            <tr>
                <td class="text-center text-white mt-5">Environment:</td>
                <td class="text-center text-white mt-5">{{ single_plant_growing.environment_type }}</td>
            </tr>
            <tr>
                <td class="text-center text-white mt-5">Starting Condition:</td>
                <td class="text-center text-white mt-5">{{ single_plant_growing.planting_type }}</td>
            </tr>
            <tr>
                <td class="text-center text-white mt-5">Water Needs:</td>
                <td class="text-center text-white mt-5">{{ single_plant_growing.water_type }}</td>
            </tr>
        </table>

    {% endblock %}

#### Custom CSS
While Bootstrap takes care of a lot, I needed to add some additional CSS to make sure my footer stuck to the bottom of the window and that text would be displayed neatly. 


## Back End

#### Relational databases
Django's ModelForm class made creating forms a breeze, but I ran into an issue where displaying an item from the database would return an error if there was no data in the child table. I used a try except block to redirect users to enter the missing data to avoid this error.

    #  View a single plant's details
    def growingDetails(request, pk):
        pk = int(pk)
        #  This page returns an error if there's no information in the GrowingDetails model,
        #  so this try/except block redirects users to the GrowingDetails entry form and provides an instruction
        try:
            single_plant_growing = GrowingDetails.objects.get(id=pk)
            context = {'single_plant_growing': single_plant_growing}
            print('success!')
            return render(request, 'GardenGuideApp/GardenGuideApp_growingDetails.html',
                          context=context)
        except ObjectDoesNotExist:
            print('There was an error')
            messages.info(request, 'Please enter growing details before continuing!')
            pass
        return HttpResponseRedirect(reverse('growing'))
        

#### Webscraping with BeautifulSoup and RegEx
I used BeautifulSoup and RegEx to find additional data on an external site about fruit and vegetable gardening. Some of the list items were linked, which threw off my regex, so I needed to write a custom function to grab the extra data.

    # View data scraped from another website
    def search(request):
        context = {}
        #  This site has data about seasonal fruits and vegetables in Oregon
        page = requests.get("https://www.thespruceeats.com/oregon-fruits-and-vegetables-2217194")
        #  BeautifulSoup object
        soup = BeautifulSoup(page.content, 'html.parser')
        #  Just the block of HTML that I want to scrape
        block = soup.find(id='mntl-sc-block_1-0-5')
        #  Regex statement to grab the formatting of the fruits
        words_i_need = block.find_all(string=re.compile(r'(\w+(\s)?):'))
        #  Get the items in the list that have links to other pages
        linked_words = block.find_all("a")

        #  Function to get the text from the linked tags
        def print_linked_words(soup_obj):
            output_list = []
            for word in soup_obj:
                output = word.get_text()
                output_list.append(output)
            return output_list

        result = print_linked_words(linked_words)
        # Check to see if 'result' stores the list correctly
        print(result)

        #  Add items to the context list to render to the page
        context['words_i_need'] = words_i_need
        context['result'] = result
        return render(request, 'GardenGuideApp/GardenGuideApp_search.html', context)

## Other Takeaways

* I took away from this project a more robust understanding of Python, a familiairity with Django's architecture, plus experience with Azure DevOps and version control as part of a team.
* Choosing to work with related tables made my work more challenging, but also forced me to understand Django's underlying structure better.
* Daily standups and weekly sprint reviews were a good introduction to DevOps culture.


