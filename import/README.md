# Import data

For the module to work, this _import_ directory must be copied to the _files_ directory of the site you are working with. I have included migration source data (the CSV files and the associated image files) in the module's directory just to be able to pack everything together and push it to a GitHub repo for making it all available to you with ease. However, the project expects the _import_ directory to be located in the _public_ directory of the site you are working with. Hence, I implemented `hook_install` to arrange the files such that, the _program.data.csv_ file is located at _public://import/program/program.data.csv_.

This is mainly because of the following reasons:

- You can place your custom import source files in one common directory, namely, _public://import_. This would allow you to place a .htaccess file in the _import_ directory and prevent HTTP access to those files.
- You code and data sources remain apart. You can change your code and push it to your code repositories without worrying about data sources provided by the client. Source data, which is often many giga bytes and subject to changes, won't get in the way of your code.
- The "modules" directories are usually not writable as a security measure. If your data sources are in a safe and writable directory, may be you can build build a UI for source data upload (CSV uploads) so that the client can upload source data on their own and re-run the migrations when required.
- For reading files from the _files_ directory of the site we can directly specify the CSV source path as _public://import/program-data/programs.csv_, however if the file were not in the _files_ directory, we would have had to do some additional coding to make the migration understand our custom path.
