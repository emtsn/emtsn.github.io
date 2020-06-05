---
layout: post
title: "Calendar App"
description: Calendar App created in Java using Swing.
date: 2019-11-28
categories: projects
tags: Java
---
![Calendar App](/assets/img-calendar-app.png)
# Calendar App created in Java using Swing
Features:
- A calendar and list display
- Single or repeating event options
- Event creator and editor
- Holidays from a [web API](https://date.nager.at/API)
- Saving and loading using [Jackson JSON Library](https://github.com/FasterXML/jackson)
- Changeable settings

Created for a course in 3 months (Sept-Nov 2019) with minor updates afterwards.

Sample code:
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

    private Display currentDisplay;
    private YearMonth selectedYearMonth;
    private LocalDate selectedDate;

    private ScheduleContainer currentSchedule;
    private Component centreComponent;

    private List<MultiEvent> currentHolidays;
    private int holidaysYear;

    private Settings currentSettings;

    public VisualEditor(String name) {
        super(name);
        initializeFields();
        initializeBaseDisplay();
        initializeSchedule();
        initializeMenu();
        showDisplay(currentDisplay);
        setVisible(true);
    }

    // MODIFIES: this
    // EFFECTS: initializes the fields
    private void initializeFields() {
        selectedYearMonth = YearMonth.from(currentDate);
        selectedDate = currentDate;
        currentSchedule = new ScheduleContainer();
        currentDisplay = Display.MonthCalendar;
        currentHolidays = new ArrayList<>();
        currentSettings = new Settings();
        currentSettings.load();
        holidaysYear = 0;
    }

    // MODIFIES: this
    // EFFECTS: initializes the schedule
    private void initializeSchedule() {
        if (currentSettings.isLoadOnStart()) {
            load();
        }
    }

    // MODIFIES: this
    // EFFECTS: initializes the base display of the frame
    private void initializeBaseDisplay() {
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setMinimumSize(new Dimension(WIDTH, HEIGHT));
        centreComponent = new JPanel();
        setResizable(false);
    }

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
    public LocalDate pickingDate() {
        if (currentDisplay.equals(Display.EventsForDate)) {
            return selectedDate;
        } else if (currentDisplay.equals(Display.MonthCalendar)) {
            return selectedYearMonth.atDay(1);
        } else {
            return currentDate;
        }
    }

    // MODIFIES: this
    // EFFECTS:: adds dateEvent to currentSchedule
    public void addEvent(DateEvent dateEvent) {
        currentSchedule.addEvent(dateEvent);
    }

    // MODIFIES: this
    // EFFECTS:: adds repeatEvent to currentSchedule
    public void addEvent(RepeatEvent repeatEvent) {
        currentSchedule.addEvent(repeatEvent);
    }

    // MODIFIES: this
    // EFFECTS: removes dateEvent from currentSchedule, show error if failed
    public void removeEvent(DateEvent dateEvent) {
        if (!currentSchedule.removeEvent(dateEvent)) {
            showError("Failed to remove event.");
        }
    }

    // MODIFIES: this
    // EFFECTS: removes repeatEvent from currentSchedule, show error if failed
    public void removeEvent(RepeatEvent repeatEvent) {
        if (!currentSchedule.removeEvent(repeatEvent)) {
            showError("Failed to remove event.");
        }
    }

    // EFFECTS: saves schedule onto saveFile
    @Override
    public void save() {
        try {
            SaveLoadSystem.saveWithJackson(currentSchedule, SAVE_FILE);
        } catch (IOException e) {
            showError("Failed to save schedule.");
        }
    }

    // MODIFIES: this
    // EFFECTS: loads schedule from saveFile
    @Override
    public void load() {
        try {
            currentSchedule = SaveLoadSystem.loadWithJackson(SAVE_FILE, ScheduleContainer.class);
            currentSchedule.sort();
            fullUpdate();
        } catch (IOException e) {
            showError("Failed to load schedule.");
        }
    }

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
    private JMenu createHelpMenu() {
        JMenu helpMenu = new JMenu("Help");
        JMenuItem showHelpMenuItem = new JMenuItem("Show Help");
        showHelpMenuItem.addActionListener(e -> showHelp());
        helpMenu.add(showHelpMenuItem);
        return helpMenu;
    }

    // MODIFIES: this
    // EFFECTS: updates the display of the editor
    public void fullUpdate() {
        showDisplay(currentDisplay);
    }

    // MODIFIES: this
    // EFFECTS: change the centre component in the editor
    private void changeCentreComponentTo(Component component, Display setDisplay) {
        getContentPane().remove(centreComponent);
        centreComponent = component;
        currentDisplay = setDisplay;
        getContentPane().add(component, BorderLayout.CENTER);
        validate();
        repaint();
    }

    // MODIFIES: this
    // EFFECTS: shows the corresponding visual display from display
    public void showDisplay(Display display) {
        switch (display) {
            case Settings:
                showSettings();
                break;
            case Help:
                showHelp();
                break;
            default:
                showViewDisplay(display);
                break;
        }
    }

    // MODIFIES: this
    // EFFECTS: shows the corresponding visual display of view displays
    private void showViewDisplay(Display display) {
        switch (display) {
            case MonthCalendar:
                showMonthCalendar();
                break;
            case EventsForDate:
                showEventsForDate(selectedDate);
                break;
            case Holidays:
                showHolidays();
                break;
            default:
                showAllEvents();
                break;
        }
    }

    // EFFECTS: returns a JLabel for a title
    private JLabel createTitleLabel(String text) {
        JLabel titleLabel = new JLabel(text, SwingConstants.CENTER);
        titleLabel.setBorder(BorderFactory.createLineBorder(Color.GRAY));
        titleLabel.setBackground(Color.WHITE);
        titleLabel.setOpaque(true);
        return titleLabel;
    }

    // MODIFIES: this
    // EFFECTS: shows the calendar display at the centre of the frame
    public void showMonthCalendar() {
        changeCentreComponentTo(createCalendarDisplay(), Display.MonthCalendar);
        currentDisplay = Display.MonthCalendar;
    }

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
    private void moveMonths(int count) {
        selectedYearMonth = selectedYearMonth.plusMonths(count);
        showMonthCalendar();
    }

    // MODIFIES: this
    // EFFECTS: shows a list of events in the current schedule
    private void showAllEvents() {
        changeCentreComponentTo(createEventsDisplay(currentSchedule, "All Events", false),
                Display.AllEvents);
    }

    // MODIFIES: this
    // EFFECTS: shows a list of events in the current schedule for the date
    private void showEventsForDate(LocalDate date) {
        changeCentreComponentTo(createEventsDisplay(currentSchedule.getScheduleForDate(date),
                "Events for " + date.format(FormatterPattern.DATE_FORMATTER), true), Display.EventsForDate);
        selectedDate = date;
    }

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
    private JLabel createEventTitle(String title) {
        JLabel titleLabel = createTitleLabel(title);
        titleLabel.setMaximumSize(new Dimension(WIDTH, titleLabel.getPreferredSize().height));
        titleLabel.setAlignmentX(Component.LEFT_ALIGNMENT);
        return titleLabel;
    }

    // EFFECTS: shows a window to create a new event
    private void showEventCreator() {
        EventCreatorFrame newEventFrame = new EventCreatorFrame(this);
    }

    // EFFECTS: shows a window to edit an event
    public void showEventEditor(EventPanel eventPanel) {
        EventEditorFrame editEventFrame = new EventEditorFrame(this, eventPanel);
    }

    // EFFECTS: create an array of booleans of length of yearMonth,
    //          where values are true if there is a event, and false if there is no event
    private boolean[] createHasEventsForYearMonth(YearMonth yearMonth) {
        boolean[] hasEvents = new boolean[yearMonth.lengthOfMonth()];
        if (currentSettings.isShowDate()) {
            setHasEventsForDateEvents(currentSchedule, yearMonth, hasEvents);
        }
        if (currentSettings.isShowRepeat()) {
            setHasEventsForRepeatEvents(currentSchedule, yearMonth, hasEvents);
        }
        return hasEvents;
    }

    // MODIFIES: hasEvents
    // EFFECTS: set values of hasEvents (array of booleans) to be true when there is a dateEvent on that dayOfMonth
    private void setHasEventsForDateEvents(ScheduleContainer schedule, YearMonth yearMonth, boolean[] hasEvents) {
        List<DateEvent> dateEventsForMonth = getDateEventsForYearMonth(schedule, yearMonth);
        for (DateEvent dateEvent : dateEventsForMonth) {
            hasEvents[dateEvent.getDate().getDayOfMonth() - 1] = true;
        }
    }

    // MODIFIES: hasEvents
    // EFFECTS: set values of hasEvents (array of booleans) to be true when there is a repeatEvent on that dayOfMonth
    private void setHasEventsForRepeatEvents(ScheduleContainer schedule, YearMonth yearMonth, boolean[] hasEvents) {
        List<RepeatEvent> repeatEvents = schedule.getRepeatEvents();
        for (int i = 0; i < yearMonth.lengthOfMonth(); i++) {
            if (!hasEvents[i]) {
                for (RepeatEvent r : repeatEvents) {
                    if (r.isOnDate(yearMonth.atDay(i + 1))) {
                        hasEvents[i] = true;
                        break;
                    }
                }
            }
        }
    }

    // EFFECTS: returns a list of all dateEvents in schedule that are in yearMonth
    private ArrayList<DateEvent> getDateEventsForYearMonth(ScheduleContainer schedule, YearMonth yearMonth) {
        List<DateEvent> dateEvents = schedule.getDateEvents();
        ArrayList<DateEvent> dateEventsForMonth = new ArrayList<>();
        boolean reached = false;
        for (DateEvent dateEvent : dateEvents) {
            if (YearMonth.from(dateEvent.getDate()).equals(yearMonth)) {
                reached = true;
                dateEventsForMonth.add(dateEvent);
            } else if (reached) {
                break;
            }
        }
        return dateEventsForMonth;
    }

    // MODIFIES: this
    // EFFECTS: shows the holiday display
    private void showHolidays() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        String holidayText = getAllHolidaysText();
        JLabel holidayListLabel = new JLabel(holidayText);
        holidayListLabel.setVerticalAlignment(SwingConstants.TOP);
        holidayListLabel.setHorizontalAlignment(SwingConstants.LEFT);
        JScrollPane scrollPane = new JScrollPane(holidayListLabel);
        mainPanel.add(createTitleLabel("Holidays for " + currentDate.getYear()), BorderLayout.PAGE_START);
        mainPanel.add(scrollPane, BorderLayout.CENTER);
        changeCentreComponentTo(mainPanel, Display.Holidays);
    }

    // EFFECTS: returns a formatted text of all of the holidays in current year
    private String getAllHolidaysText() {
        String text = "<html>";
        updateHolidays();
        for (int i = 0; i < currentHolidays.size(); i++) {
            text = text.concat(getFormattedHolidayText(currentHolidays.get(i)));
            if (i < currentHolidays.size() - 1) {
                text = text.concat("<br>");
            }
        }
        return text;
    }

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
    private void splitHolidays() {
        List<MultiEvent> newHolidays = new ArrayList<>();
        for (MultiEvent holiday : currentHolidays) {
            newHolidays.add(holiday);
            newHolidays.addAll(holiday.split());
        }
        currentHolidays.clear();
        currentHolidays = newHolidays;
    }

    // MODIFIES: this
    // EFFECTS: merge current holidays on the same date
    private void mergeHolidays() {
        for (int i = currentHolidays.size() - 1; i > 0; i--) {
            MultiEvent curr = currentHolidays.get(i);
            MultiEvent prev = currentHolidays.get(i - 1);
            if (prev.isOnDate(curr.getDate())) {
                prev.addLink(curr);
                currentHolidays.remove(i);
            }
        }
    }

    // EFFECTS: returns a list of holidays from JSONArray
    private List<MultiEvent> getHolidaysFrom(JSONArray array) {
        List<MultiEvent> retVal = new ArrayList<>();
        for (int i = 0; i < array.length(); i++) {
            retVal.add(getHolidayFrom(array.getJSONObject(i)));
        }
        return retVal;
    }

    // EFFECTS: returns a DateEvent for holiday from JSONObject
    private MultiEvent getHolidayFrom(JSONObject object) {
        LocalDate holidate = LocalDate.parse(object.get("date").toString(), DATE_DASH_FORMATTER);
        return new MultiEvent(object.get("name").toString(), holidate,
                LocalTime.of(0,0), LocalTime.of(23,59));
    }

    // EFFECTS: returns a formatted text of the holiday
    private String getFormattedHolidayText(MultiEvent holiday) {
        boolean isBefore = holiday.getDate().isBefore(currentDate);
        if (isBefore) {
            return "<font color=#999999>[" + holiday.getTimeString() + "] " + holiday.getMergedName() + "</font>";
        } else {
            return "[" + holiday.getTimeString() + "] " + holiday.getMergedName();
        }
    }

    // MODIFIES: this
    // EFFECTS: shows the setting display
    private void showSettings() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        JPanel listPanel = new JPanel();
        listPanel.setLayout(new BoxLayout(listPanel, BoxLayout.PAGE_AXIS));
        listPanel.add(createEventSettingPanel(currentSettings));
        listPanel.add(createLoadSettingsPanel(currentSettings));
        listPanel.add(createColorSettingPanel(currentSettings));
        mainPanel.add(createTitleLabel("Settings"), BorderLayout.PAGE_START);
        mainPanel.add(listPanel, BorderLayout.CENTER);
        changeCentreComponentTo(mainPanel, Display.Settings);
    }

    // EFFECTS: creates the panel for load options
    private JPanel createLoadSettingsPanel(Settings settings) {
        JPanel mainPanel = new JPanel();
        mainPanel.setLayout(new BoxLayout(mainPanel, BoxLayout.PAGE_AXIS));
        JCheckBox colorChooser = new JCheckBox("Load on Start");
        colorChooser.setSelected(settings.isLoadOnStart());
        colorChooser.addItemListener(e -> settings.setLoadOnStart(e.getStateChange() == ItemEvent.SELECTED));
        mainPanel.add(colorChooser);
        return mainPanel;
    }

    // EFFECTS: creates the panel for color options
    private JPanel createColorSettingPanel(Settings settings) {
        JPanel mainPanel = new JPanel();
        mainPanel.setLayout(new BoxLayout(mainPanel, BoxLayout.PAGE_AXIS));
        JCheckBox colorChooser = new JCheckBox("B&W mode");
        colorChooser.setSelected(settings.getHighlightColor() == Settings.DEFAULT_HIGHLIGHT_COLORLESS);
        colorChooser.addItemListener(
                e -> settings.setHighlightColor((e.getStateChange() == ItemEvent.SELECTED)
                        ? Settings.DEFAULT_HIGHLIGHT_COLORLESS : Settings.DEFAULT_HIGHLIGHT_COLOR));
        mainPanel.add(colorChooser);
        return mainPanel;
    }

    // EFFECTS: creates the panel for event options
    private JPanel createEventSettingPanel(Settings settings) {
        JPanel mainPanel = new JPanel();
        mainPanel.setLayout(new BoxLayout(mainPanel, BoxLayout.PAGE_AXIS));
        JCheckBox showDateBox = new JCheckBox("Show Date Events in Calendar");
        JCheckBox showRepeatBox = new JCheckBox("Show Repeat Events in Calendar");
        JCheckBox mergeHolidaysBox = new JCheckBox("Merge holidays on the same day");
        showDateBox.setSelected(settings.isShowDate());
        showRepeatBox.setSelected(settings.isShowRepeat());
        mergeHolidaysBox.setSelected(settings.isMergeHoliday());
        showDateBox.addItemListener(e -> settings.setShowDate(e.getStateChange() == ItemEvent.SELECTED));
        showRepeatBox.addItemListener(e -> settings.setShowRepeat(e.getStateChange() == ItemEvent.SELECTED));
        mergeHolidaysBox.addItemListener(e -> settings.setMergeHoliday(e.getStateChange() == ItemEvent.SELECTED));
        mainPanel.add(showDateBox);
        mainPanel.add(showRepeatBox);
        mainPanel.add(mergeHolidaysBox);
        return mainPanel;
    }

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
    private void showError(String text) {
        JOptionPane.showMessageDialog(this, text, "Error", JOptionPane.ERROR_MESSAGE);
    }

    public static void main(String[] args) {
        try {
            new VisualEditor("Schedule Editor");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```