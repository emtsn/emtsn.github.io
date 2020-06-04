---
layout: post
title: "Calendar App"
date: 2020-05-28
categories: projects
tags: Java
---
![Calendar App](/assets/img-calendar-app.png)
# Calendar App created in Java using Swing
Features:
- A calendar and list display
- Single or repeating event options
- Event creator and editor
- Holidays from a web API
- Saving and loading using Jackson JSON Library
- Changeable settings

Created for a course in 3 months.

Sample code (some method implementations omitted):
```java
public class VisualEditor extends JFrame implements SaveLoadSystem {
    private enum Display {
        MonthCalendar,
        AllEvents,
        EventsForDate,
        Holidays,
        Settings,
        Help,
    }

    private static final String SAVE_FILE = "data/scheduleData.json";
    private static final int HEIGHT = 500;
    private static final int WIDTH = 500;

    public static LocalDate currentDate = LocalDate.now();
    public static Color highlightColor = Color.GREEN;

    private Display currentDisplay;
    private YearMonth selectedYearMonth;
    private LocalDate selectedDate;

    private ScheduleContainer currentSchedule;
    private Component centreComponent;

    private List<MultiEvent> currentHolidays;
    private int holidaysYear;

    private Settings currentSettings;

    public VisualEditor(String name);

    // MODIFIES: this
    // EFFECTS: initializes the fields
    private void initializeFields();

    // MODIFIES: this
    // EFFECTS: initializes the schedule
    private void initializeSchedule();

    // MODIFIES: this
    // EFFECTS: initializes the base display of the frame
    private void initializeBaseDisplay();

    // MODIFIES: this
    // EFFECTS: initializes the top menu bar
    private void initializeMenu() {
        JMenuBar menuBar = new JMenuBar();
        menuBar.add(createFileMenu());
        menuBar.add(createEditMenu());
        menuBar.add(createViewMenu());
        menuBar.add(createHelpMenu());
        getContentPane().add(menuBar, BorderLayout.PAGE_START);
    }

    // EFFECTS: returns a LocalDate to set initial date editing for
    public LocalDate pickingDate();

    // MODIFIES: this
    // EFFECTS:: adds dateEvent to currentSchedule
    public void addEvent(DateEvent dateEvent);

    // MODIFIES: this
    // EFFECTS:: adds repeatEvent to currentSchedule
    public void addEvent(RepeatEvent repeatEvent);

    // MODIFIES: this
    // EFFECTS: removes dateEvent from currentSchedule, show error if failed
    public void removeEvent(DateEvent dateEvent);
    }

    // MODIFIES: this
    // EFFECTS: removes repeatEvent from currentSchedule, show error if failed
    public void removeEvent(RepeatEvent repeatEvent);

    // EFFECTS: saves schedule onto saveFile
    @Override
    public void save();

    // MODIFIES: this
    // EFFECTS: loads schedule from saveFile
    @Override
    public void load();

    // EFFECTS: creates and returns the file section of the menu bar
    private JMenu createFileMenu() {
        JMenu fileMenu = new JMenu("File");
        JMenuItem saveMenuItem = new JMenuItem("Save");
        JMenuItem loadMenuItem = new JMenuItem("Load");
        JMenuItem settingsMenuItem = new JMenuItem("Settings");
        saveMenuItem.addActionListener(e -> save());
        loadMenuItem.addActionListener(e -> load());
        settingsMenuItem.addActionListener(e -> showSettings());
        fileMenu.add(saveMenuItem);
        fileMenu.add(loadMenuItem);
        fileMenu.add(settingsMenuItem);
        return fileMenu;
    }

    // EFFECTS: creates and returns the edit section of the menu bar
    private JMenu createEditMenu() {
        JMenu editMenu = new JMenu("Edit");
        JMenuItem newMenuItem = new JMenuItem("New Event");
        newMenuItem.addActionListener(e -> showEventCreator());
        editMenu.add(newMenuItem);
        return editMenu;
    }

    // EFFECTS: creates and returns the view section of the menu bar
    private JMenu createViewMenu() {
        JMenu viewMenu = new JMenu("View");
        JMenuItem showEventsMenuItem = new JMenuItem("Events");
        JMenuItem showMonthMenuItem = new JMenuItem("Calendar");
        JMenuItem showHolidayMenuItem = new JMenuItem("Holidays");
        showEventsMenuItem.addActionListener(e -> showAllEvents());
        showMonthMenuItem.addActionListener(e -> showMonthCalendar());
        showHolidayMenuItem.addActionListener(e -> showHolidays());
        viewMenu.add(showEventsMenuItem);
        viewMenu.add(showMonthMenuItem);
        viewMenu.add(showHolidayMenuItem);
        return viewMenu;
    }

    // EFFECTS: creates and returns the help section of the menu bar
    private JMenu createHelpMenu();

    // MODIFIES: this
    // EFFECTS: updates the display of the editor
    public void fullUpdate();

    // MODIFIES: this
    // EFFECTS: change the centre component in the editor
    private void changeCentreComponentTo(Component component, Display setDisplay);

    // MODIFIES: this
    // EFFECTS: shows the corresponding visual display from display
    public void showDisplay(Display display);

    // MODIFIES: this
    // EFFECTS: shows the corresponding visual display of view displays
    private void showViewDisplay(Display display);

    // EFFECTS: returns a JLabel for a title
    private JLabel createTitleLabel(String text);

    // MODIFIES: this
    // EFFECTS: shows the calendar display at the centre of the frame
    public void showMonthCalendar();

    // EFFECTS: creates a calendar display
    private JPanel createCalendarDisplay() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        JLabel titleLabel = createTitleLabel(selectedYearMonth.format(DateTimeFormatter.ofPattern("yyyy MMMM")));
        mainPanel.add(titleLabel, BorderLayout.PAGE_START);
        mainPanel.add(createCalendar(), BorderLayout.CENTER);
        mainPanel.add(createToolPanel(e -> moveMonths(-1), e -> moveMonths(1)), BorderLayout.PAGE_END);
        return mainPanel;
    }

    // EFFECTS: creates and returns a bottom tool panel
    private JPanel createToolPanel(ActionListener leftAction, ActionListener rightAction) {
        JPanel toolPanel = new JPanel();
        toolPanel.setBackground(Color.WHITE);
        toolPanel.setBorder(BorderFactory.createLineBorder(Color.GRAY));
        if (leftAction != null) {
            JButton leftButton = new JButton("<");
            leftButton.addActionListener(leftAction);
            toolPanel.add(leftButton);
        }
        if (rightAction != null) {
            JButton rightButton = new JButton(">");
            rightButton.addActionListener(rightAction);
            toolPanel.add(rightButton);
        }
        return toolPanel;
    }

    // EFFECTS: creates and returns the main calendar panel
    private JPanel createCalendar() {
        JPanel calendarPanel = new JPanel(new BorderLayout());
        JPanel dowLabels = createDowLabels();
        JPanel monthPanel = createMonthPanel();
        calendarPanel.add(dowLabels, BorderLayout.PAGE_START);
        calendarPanel.add(monthPanel, BorderLayout.CENTER);
        return calendarPanel;
    }

    // EFFECTS: creates and returns the boxes of dates for one month
    private JPanel createMonthPanel() {
        JPanel monthPanel = new JPanel();
        monthPanel.setLayout(new GridLayout(7, 7));
        int dayOffSet = selectedYearMonth.atDay(1).getDayOfWeek().getValue();
        int lastDay = selectedYearMonth.lengthOfMonth();
        boolean[] hasEventsForDates = createHasEventsForYearMonth(selectedYearMonth);
        for (int i = 0; i < 7; i++) {
            for (int j = 0; j < 7; j++) {
                int currentIndex = (7 * i + j);
                int day = currentIndex - dayOffSet + 1;
                monthPanel.add(createDateBox(day, lastDay, hasEventsForDates));
            }
        }
        return monthPanel;
    }

    // EFFECTS: creates and returns a row of DayOfWeek labels
    private JPanel createDowLabels() {
        JPanel dowLabels = new JPanel();
        dowLabels.setLayout(new GridLayout(1,7));
        for (DayOfWeek dayOfWeek : dowValues(DayOfWeek.SUNDAY)) {
            JLabel dowLabel = new JLabel(dayOfWeek.getDisplayName(TextStyle.SHORT, Locale.getDefault()),
                    SwingConstants.CENTER);
            dowLabels.add(dowLabel);
        }
        return dowLabels;
    }

    // EFFECTS: creates and returns a box for a date
    private JButton createDateBox(int day, int lastDay, boolean[] hasEventOnDates) {
        JButton dateBox = new JButton();
        if (day >= 1 && day <= lastDay) {
            dateBox.setText("" + day);
            dateBox.setVerticalAlignment(SwingConstants.TOP);
            dateBox.setOpaque(true);
            dateBox.setBackground(Color.WHITE);
            dateBox.setBorder(BorderFactory.createLineBorder(Color.BLACK));
            dateBox.addActionListener(e -> {
                LocalDate date = selectedYearMonth.atDay(day);
                showEventsForDate(date);
            });
            if (hasEventOnDates[day - 1]) {
                dateBox.setBackground(currentSettings.getHighlightColor());
            }
        } else {
            dateBox.setVisible(false);
        }
        return dateBox;
    }

    // MODIFIES: this
    // EFFECTS: moves between the selected month to be shown
    private void moveMonths(int count);

    // MODIFIES: this
    // EFFECTS: shows a list of events in the current schedule
    private void showAllEvents();

    // MODIFIES: this
    // EFFECTS: shows a list of events in the current schedule for the date
    private void showEventsForDate(LocalDate date);

    // MODIFIES: this
    // EFFECTS: create a panel with a list of events in the schedule
    private JPanel createEventsDisplay(ScheduleContainer schedule, String titleText, boolean isDate) {
        JPanel mainPanel = new JPanel(new BorderLayout());
        JLabel titleLabel = createTitleLabel(titleText);
        mainPanel.add(titleLabel, BorderLayout.PAGE_START);
        if (isDate) {
            mainPanel.add(createToolPanel(e -> showEventsForDate(selectedDate.minusDays(1)),
                    e -> showEventsForDate(selectedDate.plusDays(1))), BorderLayout.PAGE_END);
        }
        if (schedule.getSize() < 1) {
            mainPanel.add(new JLabel("No Events...", SwingConstants.CENTER), BorderLayout.CENTER);
        } else {
            JPanel listPanel = new JPanel();
            JScrollPane scrollPane = new JScrollPane(listPanel);
            listPanel.setLayout(new BoxLayout(listPanel, BoxLayout.PAGE_AXIS));
            mainPanel.add(scrollPane, BorderLayout.CENTER);
            addEventPanels(schedule, listPanel);
        }
        return mainPanel;
    }

    // MODIFIES: listPanel
    // EFFECTS: adds EventPanels of events in the schedule to listPanel
    private void addEventPanels(ScheduleContainer schedule, JPanel listPanel) {
        if (schedule.getRepeatEventsSize() >= 1) {
            listPanel.add(createEventTitle("Repeat Events"));
        }
        for (RepeatEvent r : schedule.getRepeatEvents()) {
            EventPanel eventPanel = new RepeatEventPanel(this, r);
            eventPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
            eventPanel.setMaximumSize(new Dimension(WIDTH, eventPanel.getMaximumSize().height));
            listPanel.add(eventPanel);
        }
        if (schedule.getDateEventsSize() >= 1) {
            listPanel.add(createEventTitle("Date Events"));
        }
        for (DateEvent d : schedule.getDateEvents()) {
            EventPanel eventPanel = new DateEventPanel(this, d);
            eventPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
            eventPanel.setMaximumSize(new Dimension(WIDTH, eventPanel.getMaximumSize().height));
            listPanel.add(eventPanel);
        }
    }

    // EFFECTS: creates and returns a title JLabel for the events display
    private JLabel createEventTitle(String title);

    // EFFECTS: shows a window to create a new event
    private void showEventCreator();

    // EFFECTS: shows a window to edit an event
    public void showEventEditor(EventPanel eventPanel);

    // EFFECTS: create an array of booleans of length of yearMonth,
    //          where values are true if there is a event, and false if there is no event
    private boolean[] createHasEventsForYearMonth(YearMonth yearMonth);

    // MODIFIES: hasEvents
    // EFFECTS: set values of hasEvents (array of booleans) to be true when there is a dateEvent on that dayOfMonth
    private void setHasEventsForDateEvents(ScheduleContainer schedule, YearMonth yearMonth, boolean[] hasEvents);

    // MODIFIES: hasEvents
    // EFFECTS: set values of hasEvents (array of booleans) to be true when there is a repeatEvent on that dayOfMonth
    private void setHasEventsForRepeatEvents(ScheduleContainer schedule, YearMonth yearMonth, boolean[] hasEvents);

    // EFFECTS: returns a list of all dateEvents in schedule that are in yearMonth
    private ArrayList<DateEvent> getDateEventsForYearMonth(ScheduleContainer schedule, YearMonth yearMonth);

    // MODIFIES: this
    // EFFECTS: shows the holiday display
    private void showHolidays();

    // EFFECTS: returns a formatted text of all of the holidays in current year
    private String getAllHolidaysText();

    // MODIFIES: this
    // EFFECTS: updates the current holidays
    private void updateHolidays() {
        if (currentHolidays.size() < 1 || holidaysYear != currentDate.getYear()) {
            System.out.println("Fetching holidays...");
            String theURL = "https://date.nager.at/api/v2/PublicHolidays/" + currentDate.getYear() + "/CA";
            try {
                String webpage = PageReader.readWebPage(theURL);
                currentHolidays = getHolidaysFrom(new JSONArray(webpage));
                holidaysYear = currentDate.getYear();
            } catch (IOException | JSONException ie) {
                ie.printStackTrace();
                showError("Failed to get holidays for " + currentDate.getYear());
            }
        }
        if (currentSettings.isMergeHoliday()) {
            mergeHolidays();
        } else {
            splitHolidays();
        }
    }

    // MODIFIES: this
    // EFFECTS: split holidays on the same date
    private void splitHolidays();

    // MODIFIES: this
    // EFFECTS: merge current holidays on the same date
    private void mergeHolidays();

    // EFFECTS: returns a list of holidays from JSONArray
    private List<MultiEvent> getHolidaysFrom(JSONArray array);

    // EFFECTS: returns a DateEvent for holiday from JSONObject
    private MultiEvent getHolidayFrom(JSONObject object);

    // EFFECTS: returns a formatted text of the holiday
    private String getFormattedHolidayText(MultiEvent holiday);

    // MODIFIES: this
    // EFFECTS: shows the setting display
    private void showSettings();

    // EFFECTS: creates the panel for event options
    private JPanel createEventSettingPanel();

    // MODIFIES: this
    // EFFECTS: show help display
    private void showHelp() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        String helpText = "<html>- Use <i>File/Save</i> to save the current schedule<br>"
                + "- Use <i>File/Load</i> to load the saved schedule<br>"
                + "- Use <i>File/Settings</i> to view settings<br>"
                + "- Use <i>Edit/New Event</i> to create a new event<br>"
                + "- Use <i>View/Events</i> to view all events<br>"
                + "- Use <i>View/Calendar</i> to view the calendar<br>"
                + "&nbsp;&nbsp;- Use left/right button below to switch months<br>"
                + "&nbsp;&nbsp;- Click on a date to view the date's events<br>"
                + "- Use <i>View/Holidays</i> to view the holidays for this year";
        JLabel textLabel = new JLabel(helpText);
        textLabel.setVerticalAlignment(SwingConstants.TOP);
        textLabel.setHorizontalAlignment(SwingConstants.LEFT);
        mainPanel.add(createTitleLabel("Help"), BorderLayout.PAGE_START);
        mainPanel.add(textLabel, BorderLayout.CENTER);
        changeCentreComponentTo(mainPanel, Display.Help);
    }

    // EFFECTS: shows an error dialogue
    private void showError(String text);

    public static void main(String[] args);
}
```