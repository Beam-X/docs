# Ruby/Ruby on Rails Style Guide

## Comments

Write comments if logic in the code may not be obvious. Comments are not needed on simple and obvious things.

```ruby
# Bad

tmp = "#{Dir.tmpdir}/file.pdf"
render_file(tmp)
@record.update(document: File.open(tmp))
```

```ruby
# Good

# create temporary directory for the PDF file
temp_dir = Dir.tmpdir
temp_file = "#{temp_dir}/file.pdf"
# generate PDF file
render_file(temp_file)
# attach PDF file to the record
@record.update(document: File.open(temp_file))
```

## Quotes

Use single quotes for basic strings without special symbols. Use double quotes for strings with special symbols or interpolation.

```ruby
# Bad
"Hello Michael!"

# Good
"Hello Michael! How you're doing?"

# Good
name = "Michael"
"Hello #{name}!"
```
