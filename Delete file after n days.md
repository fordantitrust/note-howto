# Delete file after [n] days
`find . -mtime +[n] -type f -print0 | xargs -0 rm -f`

## Example 
`find . -mtime +7 -type f -print0 | xargs -0 rm -f`