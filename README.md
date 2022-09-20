# devPTIpack

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
<!-- badges: end -->

The goal of the `devPTIpack` is to be a platform that contains key
instruments for creating PTI dashboards as well as embedding PTI modules
in to the existing shiny apps.

## Installation:

    # Install development version from GitHub
    remotes::install_github("wbPTI/devPTIpack")

    # Install specific version from GitHub
    remotes::install_github("wbPTI/devPTIpack@v0.0.9")

We add `@v0.0.9` to be able to refer to a specific tag. Check the [tags’
tree](https://github.com/wbPTI/devPTIpack/tags) on github for existing
tags.

## Usage

There are several ways how this package can be used:

1.  to deploy a stand-along PTI apps to the server;

2.  to embed PTI modules into existing apps;

### Run a stand-along PTI app

PTI apps are pre-packaged in a form of two functions.

    devPTIpack::launch_pti()
    devPTIpack::launch_pti_onepage()

Minimum data preparation requirements must be met in order to launch a
PTI using functions from above. We need to have `shp_dta`, `inp_dta` and
the app’s name. In more details, data preparation is explained in
`vignette("dataprep")`. Some exemplary data sets are included in the
package: geometries `devPTIpack::ukr_shp` and metadata
`devPTIpack::ukr_mtdt_full`.

    # Running PTI app with sample data
    library(devPTIpack)
    launch_pti_onepage(
      shp_dta = ukr_shp,
      inp_dta = ukr_mtdt_full,
      app_name = "Sample PTI"
    )

Many other parameters could be specified in `devPTIpack::launch_pti()`.
The most notable are:

-   `app_name` - title of the app. Appears across multiple places in the
    app.
-   `show_waiter` - `TRUE/FALSE` indicates weather a “waiter” screen has
    to show, when the app launches.
-   `show_adm_levels` - vector `c("admin1", "admin4")` or `NULL`,
    indicates at what admin levels the map must be displayed If `NULL`
    all available administrative levels are used.
-   `shapes_path` and `mtdtpdf_path` are relative or absolute path to a
    zip archive with shape files used for mapping and metadata pdf
    respectively.
-   `wt_dwnld_options` and `map_dwnld_options` specify what data and
    metadata download options are available in different places of the
    app’s interface.

#### Deploying a stand-along PTI app

When data is prepare properly and `devPTIpack::launch_pti_onepage()`
works, we can create a dedicated folder, where app’s data and code can
be stored. This is needed if we want to deploy the PTI app to a Shiny
server or an RStudio Connect server.

To create a new PTI project use `devPTIpack::create_new_pti()`:

    devPTIpack::create_new_pti("PATH-TO-NEW-PROJECT-LOCATION/PROJECT-NAME")

-   `PATH-TO-NEW-PROJECT-LOCATION` is a relative or an absolute path to
    the folder, where the app should be created.

-   a new sub-folder with the app itself will be created with the named
    `PROJECT-NAME`.

This is how a typical PTI folder looks like.

    +-- app-data
    |   \-- metadata.Rmd
    +-- app.R
    +-- app-name.Rproj
    +-- landing-page.md
    \-- R
    |   \-- _disable_autoload.R

After the basic app folder was created, users need to populate it with
data and make some minor edits.

-   Step 1. Prepare proper `metadata.xlsx` and `shapes.rds` files; Files
    names can be anything. One will have to specify correct paths to the
    files in `app.R`.

-   Step 2. Place shapes and metadata into the `app-data` folder. Use
    the `app-data` folder to store also other data such as archive with
    the geometrical boundaries or metadata pdf documentation.

-   Step 3. Knit `metadata.Rmd` in order to generate `metadata.pdf`
    documentation of the data used in the app. This documentation is
    based on the `metadata.xlsx` and `shapes.rds`. Remember to re-render
    metadata PDF whenever the metadata changes.

-   Step 4. Modify the `app.R` file specifying paths to `metadata.xlsx`,
    `shapes.rds`, pdf documentation and archive with the geometrical
    boundaries. Change the app’s name and tune
    `devPTIpack::launch_pti()` function according to your needs.

    Note, that `inp_dta` parameter in `devPTIpack::launch_pti()` shell
    be specified as a pre-loaded r object. As it is usually generated
    based on the `metadata.xlsx`, use
    `devPTIpack::fct_template_reader()` (see
    `?devPTIpack::fct_template_reader`).

-   Step 5. Run the app and troubleshoot its functioning. Remember, if
    the app does not start, it is likely that there are some
    discrepancies in the data preparation. Always use the built-in data
    to check if the app launches from its own folder. You may try using
    `devPTIpack::validate_geometries()` or
    `devPTIpack::validate_metadata()` to check if geometries or metadata
    are prepared properly. These validation functions, however, are not
    exhaustive, thus, consult `vignette("dataprep")` or ask for help.

-   Step 6. Develop additional functionality in folder `R`.

-   Step 7. `landing-page.md` is an optional parameter and may be used
    for customizing the info modal that pops-up when the app launches.

Fully, completed app folder may look like this:

    +-- app-data
    |   +-- admin_bounds.rds
    |   +-- metadata.pdf
    |   +-- metadata.Rmd
    |   +-- mtdt.xlsx
    |   \-- shapefiles.zip
    +-- app.R
    +-- examplePTIapp.Rproj
    +-- landing-page.md
    \-- R
        \-- _disable_autoload.R

The `app.R` script, which launches the app and could be used to deploy
such app to an external server is:

    # Launch the ShinyApp (Do not remove this comment)
    # To deploy, run: rsconnect::deployApp()
    # Or use the blue button on top of this file

    library(shiny)
    library(devPTIpack)

    launch_pti(
      shp_dta = readRDS("app-data/admin_bounds.rds"),
      inp_dta = devPTIpack::fct_template_reader("app-data/mtdt.xlsx"),
      app_name = "Example country", 
      show_waiter = TRUE, 
      show_adm_levels = c("admin1", "admin2"),
      shapes_path = "app-data/shapefiles.zip", 
      mtdtpdf_path = "app-data/metadata.pdf",
      pti_landing_page = "./landing-page.md"
    )

### Embedding PTI modules into existing apps;

Essentially, PTI is a set of modules, which are used in different
combinations. `devPTIpack::launch_pti_onepage()` offers simple glance at
one key meta-module that could be used to embed PTI into any Shiny app.

    # Server side
    ?devPTIpack::mod_ptipage_newsrv

    # UI side
    ?devPTIpack::mod_ptipage_box_ui
    ?devPTIpack::mod_ptipage_twocol_ui

`devPTIpack::mod_ptipage_newsrv()` could be paid with any of two
available UI and launched as an app in the command line. Note that in
order for the module to function with its UI, they have to be identified
in the same namespace provided with `id` parameter. For more information
on modules, see [19 Shiny
modules](https://mastering-shiny.org/scaling-modules.html) in [Mastering
Shiny](https://mastering-shiny.org/index.html) or [Modularizing Shiny
app code](https://shiny.rstudio.com/articles/modules.html) or [10
Building the App with
{golem}](https://engineering-shiny.org/build-app-golem.html#build-app-golem).

    library(shiny)
    library(devPTIpack)

    # user interface
    ui <- mod_ptipage_twocol_ui(id = "pti_mod")

    # server
    server <- function(input, output, session) {
      mod_ptipage_newsrv(id = "pti_mod",
                         inp_dta = reactive(ukr_mtdt_full),
                         shp_dta = reactive(ukr_shp), 
                         show_waiter = TRUE)
      }

    # Shiny app.
    shinyApp(ui, server)

Here is another example of embedding the PTI into an existing dashboard.
It should be runnable in the console if all necessary packages are
installed.

-   Note that most of the code below describe the dashboard layout and
    structure and only a few lines are dedicated to the PTI.
-   Remember to disable waiter when embedding module in a multitab
    dashboard.
-   More elaborate example of multitab dashboard is in the source code
    of the function `devPTIpack::launch_pti`.

<!-- -->

    ## app.R ##
    library(shiny)
    library(shinydashboard)
    library(devPTIpack)

    ui <- dashboardPage(
      dashboardHeader(title = "Basic dashboard"),
      ## Sidebar content
      dashboardSidebar(
        sidebarMenu(
          menuItem("Dashboard", tabName = "dashboard", icon = icon("dashboard")),
          menuItem("PTI", tabName = "pti", icon = icon("th"))
        )
      ),
      ## Body content
      dashboardBody(
        tabItems(
          # First tab content
          tabItem(tabName = "dashboard",
            fluidRow(
              box(plotOutput("plot1", height = 250)),

              box(
                title = "Controls",
                sliderInput("slider", "Number of observations:", 1, 100, 50)
              )
            )
          ),

          # Second tab content
          tabItem(tabName = "pti",
            fluidRow(
              box(
                
                ### ### ### ### ### ### ## ### ### ### ### ###
                ### ### ### ### ### PTI UI ### ### ### ### ###
                ### ### ### ### ### ### ## ### ### ### ### ###
                devPTIpack::mod_ptipage_box_ui(
                  id = "pti_mod", 
                  map_height = "575px", 
                  side_width = "325px",
                  wt_style = "zoom:0.8;"
                  ),
                width = 10,
                height = "600px"
                )
            )
          )
        )
      )
    )

    server <- function(input, output) {
      set.seed(122)
      histdata <- rnorm(500)

      output$plot1 <- renderPlot({
        data <- histdata[seq_len(input$slider)]
        hist(data)
      })
      
      
      ### ### ### ### ### ### ## ### ### ### ### ###
      ### ### ### ### ### PTI SERVER ### ### ### ###
      ### ### ### ### ### ### ## ### ### ### ### ###
      mod_ptipage_newsrv(
        id = "pti_mod",
        inp_dta = reactive(ukr_mtdt_full),
        shp_dta = reactive(ukr_shp), 
        show_waiter = TRUE
        )
      }

    shinyApp(ui, server)
