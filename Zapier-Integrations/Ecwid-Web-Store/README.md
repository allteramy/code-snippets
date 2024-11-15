# Ecwid Web Store Integration - Zapier Python Code

For this project, we needed to take in the Ecwid sale data and extract purchase information for a single product in order to track each product sale separately in Salesforce as its own opportunity record. The order number would be added to the opportunity so that orders with multiple products could still be tracked collectively via reports.

#### Systems: Ecwid | Zapier | Salesforce

## Zap setup:


### Steps 1 through 4 

These steps involved taking in the store order and filtering for the specific product number (00002 in this example - a resource guide).
<br>
<br>


![alt text](./assets/01-Ecwid-CSAGuide-Steps1-4.jpg "Steps 1-4")

___
### Step 5 - Determine Product Quantities & Options

![alt text](./assets/02-CSAGuide-DetermineOptions.jpg "Zap web listing")

Step 5 Code (Python)
```python
import json

# This code tests for different quantities and scu numbers to find the one we're looking for 
#   (In this partular Zap - '00002' or the CSA Guide)
#       Sample quantity q_list = ['1', '3', '2', '4']
#       Sample sku_list = ['00001', '00003', '00003', '00003']

# Steps: split input_data into three separate lists for quanity, sku, and options
#   Here's an example of what input_data would look like:
#       input_data = {'sku': '00003,00003', 'quantity': '1,1'}

q_list = list(input_data.get('quantity').split(","))
sku_list = list(input_data.get('sku').split(","))
opt_list = list(input_data.get('options').split(","))

# Dynamically set up a key to equal entry, and use a for loop to add in which 
# quantity based on the instance of finding a scu 00002
#   It will keep track of which one we're on (how many we've found)
#   Each will be stored in the json formatted string as Quantity1, Quantity2

q_index = 'Quantity'

rstring = ''          # Initialize the return string to parse into json loads

entries = {}          # If you return entries, it works, but we want to return size too

size = 0              # We want to know how many 00002 items were purchased
                      #     within each category - 5 Pack, 1 Guidebook
                      #     so we can use that to add multiple products by pricebook entry ids

i = 1                 # Keep track of which 00002 entry we're on. 

# Loop through each list and for each 00002 scu we find, add an entry# to the string
#   and increase the size and entry# (i) counters

for x in range(len(q_list)):
    if sku_list[x] == '00002':
        q_index = q_index + str(i)
        entries[q_index] = q_list[x]
        if size >= 1:
            rstring = rstring + ', \"' + q_index + '\"' + ': ' + '\"' + q_list[x] + '\"'
        else:
            rstring = rstring + '\"' + q_index + '\"' + ': ' + '\"' + q_list[x] + '\"'
        q_index = 'Quantity'
        size += 1
        i += 1

# Create a json-formatted string for size to concatenate to the main return string
#   use double quotes - was not working with single quotes

size_str = '\"size\"' + ': ' + '\"' + str(size) + '\" '

# Create our return string which must be in a json serializable format.
#   This example string is correct: '{"entry1": "1", "entry2": "1"}'

y = '{' + rstring + ', ' + size_str + '}'

# Use json.loads to format the string to be json seriazable and recognized 
#   by the return function
z = json.loads(y)

# print(type(z))
# print(type(entries))

return z

#return entries    //works, but we wanted to return size too, so we craft a customized string y above

#version: v1. "V2" is in use by Clery C. Jun 29, 2023

```

Sample Input:

![alt text](./assets/03-Product-Input.jpg "Data Input - Ecwid Order")

Sample Output:

![alt text](./assets/04-ProductQuantity-Output.jpg "Data Output - Code")



___


### Steps 6 through 8 

Here we check if the order is paid and determine the payment method and pricebook entry IDs.

<br>

![alt text](./assets/05-Steps6-8.jpg "Steps 6-8")
___

### Step 6 - Determine Payment Status

<br>

![alt text](./assets/06-Step6-Setup.jpg "Step 6 setup/input")

Step 6 code:

```python

print(input_data)

if 'PAID' in input_data.values():
    status = 'True'
else:
    status = 'False'

return{
    'status': status if status else 'ERROR'
}

```

Sample Input:

![alt text](./assets/06-Step6-Input.jpg "Step 6 input")

Sample Output:

This code correctly returns "False" since the payment status is "AWAITING_PAYMENT."

![alt text](./assets/06-Step6-Output.jpg "Step 6 output")
___

### Step 7 - Determine Payment Method
<br>

![alt text](./assets/07-Step7-Setup.jpg "Step 7 Setup")





