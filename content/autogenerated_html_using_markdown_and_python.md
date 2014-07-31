Title: auto-generated HTML using markdown and python
Date: 2012-02-01 10:20
Category: blog
Tags: python, markdown

This is my new site that has been build for speed and easy of update.
I wanted to the source files in markdown ( or something) so that could edit
the files with out a hassle and them have them compiled into HTML.

## The advantage

- Browsers and web servers to take more advantage of caching, speeding up the site.
- The pages are not generated on the server when a user requests the page, that
have all been generated way before render. This speeds up the site and reduces the overhead of
a hit on the server.
- I can update my site using markdown via vim without writing HTML.
- All the pages are forced to look uniformed as all the formatting is forced into css.
- No more wordpress, no more php script kiddy's.
- I get to use more python.


## The code

Most of the heavy lifting is completed by the below lines code code.

```python

def generate_page(page_file, header_html, footer_html, root):
    """
    Take a markdown page name, a header, a footer and its location and
    turn it into a full html page, ready to be uploaded to a remote system.
    """
    # Remove the .md file extension from the page_file into the page_name
    page_name = page_file.replace('.md','')
    # Set the output_dir local varible from the settings file
    output_dir = settings.output_dir
    # generate the path to the page based on the root and page_name
    page_path = '%(root)s/%(page_file)s' % locals()
    # Generate the directory where the complied html will end up 
    dir_target = '%(output_dir)s/%(root)s' % locals()
    # Remove the input_dir from the output target path
    dir_target = dir_target.replace('/src','')
    # Generate the location where the complied html will end up 
    page_target = '%(dir_target)s/%(page_name)s.html' % locals()
    # Make sure the output directory is on the system
    if not os.path.exists( dir_target ):
        os.mkdir( dir_target )
    # Read in the source page ( as markdown )
    page_markdown = open( page_path ).read()
    # Create a markdown2 object
    markdowner = markdown2.Markdown(extras=["fenced-code-blocks", \
        "fenced_code", "pyshell"])
    # Convert the markdown into html
    page_html = markdowner.convert( page_markdown )
    # Write the file into the target area.
    f = open( page_target , 'w')
    f.write( header_html )
    f.write( page_html )
    f.write( footer_html )
    f.close()

```
