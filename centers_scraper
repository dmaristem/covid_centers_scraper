from requests import get
from requests.exceptions import RequestException
from contextlib import closing
from bs4 import BeautifulSoup, Tag, NavigableString
import re
import csv
from typing import List, Dict, Optional, Any

# Download web pages to get the raw HTML, with the help of the requests package
def simple_get(url:str):
    """
    Attempts to get the content at 'url' by making an HTTP GET request.
    If the content-type of response is some kind of HTML/XML, return the text content, otherwise return None.
    """
    try:
        #The closing() function ensures that any network resources are freed when they go out of scope in the with block.
        #Using closing() is a good practice to help prevent fatal errors and network timeouts
        with closing(get(url, stream=True)) as resp:
            if is_good_response(resp):
                print("HTTP Error: {0}".format(resp.raise_for_status()))
                print(resp.headers)
                #the content is the HTML document
                return resp.content
            else:
                return None

    except RequestException as e:
        log_error('Error during requests to {0} : {1}'.format(url, str(e)))
        return None


def is_good_response(resp)-> bool:
    """
     Return True if the response seems to be HTML, otherwise return False.
    """
    content_type = resp.headers['Content-Type'].lower()
    print("HTTP Status Code: {0}".format(resp.status_code))
    return (resp.status_code == 200 and content_type is not None and content_type.find('html') > -1)


def log_error(e):
    """
    This function prints the errors.
    """
    print(e)


def get_centers(url: str) -> None:
    """
    Create a CSV file containing the covid testing centers' data in Toronto.
    i.e. center name, address, hours, phone number, website, accessibility, etc.
    """

    response = simple_get(url)
    csv_data = []

    if response is not None:
        html = BeautifulSoup(response, 'html.parser')
        toronto = html.find("div", attrs={"class": "accordion__item"})
        toronto_centers = toronto.find_all("div", attrs={"class": "ontario-assessment-centre-card__wrapper"})
        for centers in toronto_centers: # by card
            center_name = centers.find("p", attrs={"class": "ontario-assessment-centre__title"}) # center name
            address_lines = center_name.find_next_siblings()
            address = ''
            loop_count = 1
            data_dict = {}

            # Scrape the center name and address
            for line in address_lines:
                if loop_count == 1:
                    match = re.findall("\A[a-zA-Z]", line.get_text(separator="\n"))
                    if match:
                        center_name = center_name.get_text() + "/" + line.get_text(separator="\n")
                    else:
                        address += line.get_text(separator="\n")
                        center_name = center_name.get_text(separator="\n")
                else:
                    match = re.findall("\d\Z", line.get_text(separator="\n").strip())
                    if match:
                        address += line.get_text(separator="\n")
                        break
                    address += line.get_text(separator="\n")
                loop_count += 1
            data_dict['center_name'] = center_name
            data_dict['address'] = address

            # Create key-value entries for unchanging titles i.e. Hours, Phone number, Website, etc.
            hours = centers.find("p", string="Hours").find_next_sibling().get_text(separator="\n")
            data_dict['hours'] = hours
            try:
                phone = centers.find("p", string="Phone number").find_next_sibling().get_text(separator="\n")
            except AttributeError:
                phone = ''
            data_dict['phone'] = phone
            try:
                website = centers.find("p", string="Website").find_next_sibling().get_text(separator="\n")
            except AttributeError:
                website = ''
            data_dict['website'] = website
            try:
                accessibility = centers.find("p", string="Accessibility").find_next_sibling().get_text(separator="\n")
            except AttributeError:
                accessibility = ''
            data_dict['accessibility'] = accessibility
            try:
                visitors = centers.find("p", string="Who can get a test").find_next_sibling().get_text(separator="\n")
            except AttributeError:
                visitors = ''
            data_dict['visitors'] = visitors
            try:
                details = centers.find("p", string="Appointment and location details").find_next_sibling().get_text(separator="\n")
            except AttributeError:
                details = ''
            data_dict['details'] = details
            try:
                unit = centers.find("p", string="Public Health Unit").find_next_sibling().get_text(
                    separator="\n")
            except AttributeError:
                unit = ''
            data_dict['unit'] = unit

            csv_data.append(data_dict)

        # Create CSV file with scraped data
        keys = csv_data[0].keys()
        with open('to_covid_centers.csv', 'w', newline='') as output_file:
            dict_writer = csv.DictWriter(output_file, keys)
            dict_writer.writeheader()
            dict_writer.writerows(csv_data)




if __name__ == '__main__':
    get_centers("https://covid-19.ontario.ca/assessment-centre-locations/")

# Map it find_next_siblings question - why did this function not pick up Map it? a tag. It picked up ul tag in other uses.
