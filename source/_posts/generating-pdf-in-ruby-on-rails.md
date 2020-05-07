---
title: Generating Pdf in Ruby on Rails
date: 2020-02-26 06:02:58
author: Ashish Jajoria
categories:
- ["Ruby on Rails"]
tags: 
- backend
- rails
- ruby
- ror
- pdf
- generatepdf
- Ashish_Jajoria
---

We all must have got requirement to generate PDFs at backend and store that on cloud. Well, here is a quick guide on how you can start generating the PDFs your own way without any limits.

{% asset_img prawn.png %}

## Lets start generating PDF step by step:-

1: Add **prawn** gem to your Gemfile

```ruby
gem 'prawn'
```

2: Create an instance of PDF document

```ruby
receipt_pdf = Prawn::Document.new
```

3: Draw some text and style that in your own way

![Text Styling](text_style.png)

```ruby
receipt_pdf = Prawn::Document.new
receipt_pdf.text 'My Text'
receipt_pdf.text 'My Styled Text', style: :bold
receipt_pdf.text 'My Sized Text', size: 20
receipt_pdf.text 'My Colored Text', color: '7f7f7f'
receipt_pdf.text 'My Aligned Text', align: :right
receipt_pdf.render_file 'my_pdf_file.pdf'
```

4: Adding a space/gap before and after drawing a text

![Text Gapping](text_spacing.png)

```ruby
receipt_pdf = Prawn::Document.new
receipt_pdf.text 'My Text'
receipt_pdf.move_down 50
receipt_pdf.text 'My Text After Moving down'
receipt_pdf.render_file 'my_pdf_file.pdf'
```

5: Generate Output file

```ruby
# This will create a file at your project root directory
receipt_pdf.render_file 'my_pdf_file.pdf'
```

## Drawing tables:-

1: Prepare data to draw the table

```ruby
# Prepare receipt details to show in a table in the following format
# [[a1,a2]
# [b1,b2]]
#
# this will form following table
# __|_1___2__
# a | a1  a2
# b | b1  b2

table_data = [['Items', 'Rates'],
              ['Item1', "1"],
              ['Item2', "2"],
              ['', ''], # For adding gap between my data
              ['Item3', "3"],
              ['Item4', "4"]]
```

2: Add **prawn/table** requirement before drawing the table

```ruby
require 'prawn/table'
```

3: Draw Table using prepared data

![Text Gapping](table.png)

```ruby
receipt_pdf = Prawn::Document.new
receipt_pdf.table table_data # table_data used from previous step
receipt_pdf.render_file 'my_pdf_file.pdf'
```

4: Styling Rows/Columns/Cells

![Table Styling](table_formatting.png)

```ruby
receipt_pdf = Prawn::Document.new

receipt_pdf.table table_data, cell_style: {border_width: 0, width: 250, padding: [5, 0, 5, 0], text_color: '373737', inline_format: true} do
 # Aligning a specific column cells' text to right
 columns(-1).align = :right

 # To add bottom padding to a specific row
 row(-2).padding_bottom = 10

 # To set width of border for a specific row
 row(-1).border_top_width = 1
end

receipt_pdf.render_file 'my_pdf_file.pdf'
```

## Adding custom font to your PDF document

1: Download Font Files in **.ttf** format
2: Place them into *font/your_font_name* directory at project root level(Create one if it's not there)
3: Set that font to your PDF document instance

![Using Cutom Font](text_font.png)

```ruby
your_font = 'font/your_font_name/your_font_name.ttf'
receipt_pdf = Prawn::Document.new

default_font = receipt_pdf.font.name

receipt_pdf.text 'Default Font Text'

# This will change font of your entire document after setting this
receipt_pdf.font your_font

receipt_pdf.text 'Your Font Text'

# This will change font of your entire document to default font
receipt_pdf.font default_font

receipt_pdf.move_down 30

# To Style all cells of a table
receipt_pdf.table table_data, cell_style: {font: your_font}

receipt_pdf.move_down 30

# To Style all cells of a specific column
receipt_pdf.table table_data do
 columns(-1).font = your_font
end

receipt_pdf.render_file 'my_pdf_file.pdf'
```

## Limit the PDF page size to drawn area only OR Remove extra white area after drawing all your data

[my_pdf_file.pdf](my_pdf_file.pdf)

1: Set page length to much higher value than what you want to draw while creating instance on your PDF document

```ruby
receipt_pdf = Prawn::Document.new(page_size: [600, 2000], margin: 50)
```

2: Do your drawing

```ruby
receipt_pdf.text 'My Drawing Here'
```

3: Clip or Resize the document to the drawn area

```ruby
padding_after_drawing = 15
initial_width = 600
initial_height = 2000

receipt_pdf.page.dictionary.data[:MediaBox] = [0, receipt_pdf.y - padding_after_drawing, initial_width, initial_height]

receipt_pdf.render_file 'my_pdf_file.pdf'
```

## References:-

1. [Prawn Guide](http://prawnpdf.org/manual.pdf) with examples
2. [Prawn Table Guide](http://prawnpdf.org/prawn-table-manual.pdf) with examples
