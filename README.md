# Regu-Genius

**Steps to recreate and test this model**

**Step1: Finetuning**

Head over to https://colab.research.google.com/drive/1YawxYi6rww8aVY3LcaGxbu9MRw-vX1M8?authuser=1#scrollTo=-4rzM1KEaNv- or RegGenisv2.ipynb

Re-run from section __Dataset For Finetuning__ to __Creating Finetuning Job__ a finetuned model id will be generatd

**Step2: Testing**

# To Interact directly with the model, below is a sample function.

**The generated model id is required a sample is=> ft:gpt-3.5-turbo-0613:personal:regugen:8TyfLt5J**

OPENAIKEY api key is required for every prompts

``` python code
def prompt_fm(NLT_Rules):
    # function that submit prompts to model and returns a json rule
    fine_tuned_model_id = "ft:gpt-3.5-turbo-0613:personal:regugen:8TyfLt5J"
    openai.api_key= <OPENAIKEY>
    response = openai.ChatCompletion.create(
    model=fine_tuned_model_id, 
    messages=[
        {"role": "system", "content": "You are an assistant for JSON Generation of Rules"},
        {"role": "user", "content": NLT_Rules}]
    , temperature=0
    )
    fm_res=response["choices"][0]["message"]["content"]
    print(fm_res)
    return fm_res;
```

**Where NLT_Rules is the extrated Natural Language Text from the HTML RASE tagged document**

**Sample Preprocessing**

``` python code
 NLT_Rules = extractNLTRules(RASETAGGED_Rules)
```

``` python code
output_data = StringIO()

def replace_whitespace(text):
    # A function that replace consecutive whitespaces with a single space
    return re.sub(r'\s+', ' ', text)


def extract_requirement_sections(tag):
    #
    #  A recursive function that extract requirements from each section
    # Check if the current tag is a Tag and has the "data-rasetype" attribute with the value "RequirementSection"
    #
    if isinstance(tag, Tag) and tag.get("data-rasetype") == "RequirementSection":

        # If found, store the content of the tag
        output_data.write("REQ: "+ replace_whitespace(tag.get_text())+ "\n")
    elif isinstance(tag, Tag):
        # If the current tag is a Tag, recursively call the function for each child of the current tag
        for child in tag.children:
            if(child.name != 'section' ):
                extract_requirement_sections(child)


def extract_section_info(section, d, soup):
    # A recursive function that extract Applications and Requirements(rules) from each sections keeping the hirachy
    baseSec = soup.find('section', {'title': section.get('title')})

    if(d==1):
        parghs = baseSec.find_all('p', recursive=False)
        for pargh in parghs:
            extract_requirement_sections(pargh)

    # check content
    subSecs= baseSec.find_all('section', recursive=False)
    for sec in subSecs:
        output_data.write("\t"*d +sec.get('title')+":"+ "\n")
        # extract_cont(sec,d, True)
        extract_requirement_sections(sec)

        extract_section_info(sec,d+1, soup)

def extractNLTRules(html_content) -> str:
    soup = BeautifulSoup(html_content, 'html.parser')
    # Finding all the first node section tags to get sections from the html input
    sections = soup.find_all('section', limit=1)
    # Going through each of these sections to extract the rase tag applocations and rules/requirements
    for section in sections:
        output_data.write(section.get('title')+":"+ "\n")
        extract_section_info(section,1,soup)

    # Get the captured output as a string
    output_data.seek(0)
    output = output_data.read()
    # print("output:", output)
    # Closing StringIO
    output_data.truncate(0)
    output_data.seek(0)
    return output

```


