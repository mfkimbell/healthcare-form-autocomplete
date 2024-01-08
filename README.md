# healthcare-form-autocomplete

# I currently maintain this script for a hospital in a separate private repository. In this public version, I have removed all references to the source and will only be discussing code snippets and project structure

### **Tools Used:**
* `selenium` python package for automated web interaction
* `pandas` for extracting excel data
* `pipenv` for managing dependencies and virtual enviornments

### **Things I Learned**
* with selenium, many times you have to dictate scrolling to allow objects to be visible
* when pages load, you have to pause the script to allow content to load
* some html objects can block others, so you have to be careful when, how, and in what order you interact with content
* allowing staff to have a "test" version of the code that refreshes the pages instead of submitting subsequent forms is helpful for book keeping purposes

I have various functions built like this that handle different data inputs forms might encounter:

``` python

def fillBasicSelect(input, xpath):
    select = Select(driver.find_element_by_xpath(xpath))
    select.select_by_visible_text(input)


def fillAdvancedSelect(input, xpath, innerXpath):
    inputBox = driver.find_element_by_xpath(xpath)
    inputBox.click()
    inputInnerBox = driver.find_element_by_xpath(innerXpath)
    time.sleep(0.2)
    inputInnerBox.send_keys(input)
    driver.find_element_by_xpath(innerXpath).send_keys(Keys.ENTER)

... and so on
```
Some of them are more advanced, like ones that need to account for slightly differing duplicates of the same entry codes: 

``` python
def selectCase(code: str, type: str) -> None:
    print("type before: ", type)
    type = newInput(type)
    print("type after: ", type)
    elements = driver.find_elements_by_xpath(
        "" + code + ""
    )
    for element in elements:
        print(element.text)

        # Find the parent <tr> element of the initial element
        parent_row = element.find_element_by_xpath("")
        driver.execute_script("arguments[0].scrollIntoView(true);", parent_row)
        time.sleep(0.5)
        print("Parent Row:", parent_row.text)

        td_element = parent_row.find_element_by_xpath("")
        print("td_element", td_element.text)

        # Find the "Add" button in the last column of the same row
        add_button = parent_row.find_element_by_xpath("")
        print("Add Button Text:", add_button.text)

        print("td: ", td_element.text)
        print("type", type)
        if td_element.text == type:
            
            driver.execute_script("arguments[0].scrollIntoView(true);", add_button)
            time.sleep(0.5)
            driver.execute_script("arguments[0].click();", add_button)
         
        time.sleep(0.5)

... and so on
```

To grab data, I used pandas to gather the info in a class called "Case". I then return those objects in a function from another file:

``` python
cases = grabCases()

i = 0
for case in cases:
    i += 1
    print(f"case {i}:", case)
    dateString = case.CaseDate.strftime("%m-%d-%Y")
    print(dateString)

    driver.execute_script(
        "arguments[0].scrollIntoView(true);",
        driver.find_element_by_xpath(''),
    )
    time.sleep(0.5)

    fill(case.LesionID, '')

    fillCaseDate(dateString, '')

... and so on
```
I have a shell script I have them run:
```
#!/bin/zsh

# Install dependencies from requirements.txt
pip install -r requirements.txt

# Activate virtual environment directly
source $(pipenv --venv)/bin/activate

# Run your Python script
python formFill.py
```

And I give them the following instructions:
```
Run these commands in your terminal (only necessary for the first use):

chmod +x run.sh

chmod +x test.sh

After that, run these two commands to run and test:

./run.sh

./test.sh
```
And here are all the files:

<img width="301" alt="Screenshot 2024-01-08 at 3 01 56 PM" src="https://github.com/mfkimbell/healthcare-form-autocomplete/assets/107063397/4eb674a8-3608-4e1e-b2f8-5fca98e65dd4">

