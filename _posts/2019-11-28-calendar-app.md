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

    public static final int HEIGHT = 500;
    public static final int WIDTH = 500;

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
        File file = new File(Settings.DEFAULT_DIRECTORY);
        file.mkdirs();
        System.out.println(file.getAbsolutePath());
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
        if (currentSettings.isLoadOnStart() && !currentSettings.getSaveFile().isEmpty()) {
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

    // MODIFIES: this
    // EFFECTS: changes the saveFile of the current settings
    public void setSaveFile(String fileName) {
        currentSettings.setSaveFile(fileName);
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
            SaveLoadSystem.saveWithJackson(currentSchedule, currentSettings.getSaveFile());
        } catch (IOException e) {
            showError("Failed to save schedule. Please check the save file location in settings.");
        }
    }

    // MODIFIES: this
    // EFFECTS: loads schedule from saveFile
    @Override
    public void load() {
        try {
            currentSchedule = SaveLoadSystem.loadWithJackson(currentSettings.getSaveFile(), ScheduleContainer.class);
            currentSchedule.sort();
            fullUpdate();
        } catch (IOException e) {
            showError("Failed to load schedule. Please check the save file location in settings.");
        }
    }

    // MODIFIES: this
    // EFFECTS: brings up prompt to choose a file location
    public void openFilePrompt() {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setFileFilter(new FileNameExtensionFilter("JSON Files", "json"));
        fileChooser.setFileSelectionMode(JFileChooser.FILES_ONLY);
        int option = fileChooser.showDialog(this, "Select");
        if (option == JFileChooser.APPROVE_OPTION) {
            File file = fileChooser.getSelectedFile();
            currentSettings.setSaveFile(file.getAbsolutePath());
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

    // MODIFIES: this
    // EFFECTS: shows the calendar display at the centre of the frame
    public void showMonthCalendar() {
        changeCentreComponentTo(createCalendarDisplay(), Display.MonthCalendar);
        currentDisplay = Display.MonthCalendar;
    }

    // EFFECTS: creates a calendar display
    private JPanel createCalendarDisplay() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        mainPanel.add(new TitleLabel(selectedYearMonth.format(DateTimeFormatter.ofPattern("yyyy MMMM"))),
                BorderLayout.PAGE_START);
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
        boolean[] hasEventsForDates = currentSchedule.hasEvents(selectedYearMonth,
                currentSettings.isShowDate(), currentSettings.isShowRepeat());
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
        mainPanel.add(new TitleLabel(titleText), BorderLayout.PAGE_START);
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
            listPanel.add(new TitleLabel("Repeat Events"));
        }
        for (RepeatEvent r : schedule.getRepeatEvents()) {
            EventPanel eventPanel = new RepeatEventPanel(this, r);
            eventPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
            eventPanel.setMaximumSize(new Dimension(WIDTH, eventPanel.getMaximumSize().height));
            listPanel.add(eventPanel);
        }
        if (schedule.getDateEventsSize() >= 1) {
            listPanel.add(new TitleLabel("Date Events"));
        }
        for (DateEvent d : schedule.getDateEvents()) {
            EventPanel eventPanel = new DateEventPanel(this, d);
            eventPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
            eventPanel.setMaximumSize(new Dimension(WIDTH, eventPanel.getMaximumSize().height));
            listPanel.add(eventPanel);
        }
    }

    // EFFECTS: shows a window to create a new event
    private void showEventCreator() {
        EventCreatorFrame newEventFrame = new EventCreatorFrame(this);
    }

    // EFFECTS: shows a window to edit an event
    public void showEventEditor(EventPanel eventPanel) {
        EventEditorFrame editEventFrame = new EventEditorFrame(this, eventPanel);
    }

    // MODIFIES: this
    // EFFECTS: shows the holiday display
    private void showHolidays() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        updateHolidays();
        String holidayText = HolidayUtility.getFormattedHolidaysText(currentHolidays, currentDate);
        JLabel holidayListLabel = new JLabel(holidayText);
        holidayListLabel.setVerticalAlignment(SwingConstants.TOP);
        holidayListLabel.setHorizontalAlignment(SwingConstants.LEFT);
        JScrollPane scrollPane = new JScrollPane(holidayListLabel);
        mainPanel.add(new TitleLabel("Holidays for " + currentDate.getYear()), BorderLayout.PAGE_START);
        mainPanel.add(scrollPane, BorderLayout.CENTER);
        changeCentreComponentTo(mainPanel, Display.Holidays);
    }

    // MODIFIES: this
    // EFFECTS: updates the current holidays
    private void updateHolidays() {
        if (currentHolidays.size() < 1 || holidaysYear != currentDate.getYear()) {
            String theURL = "https://date.nager.at/api/v2/PublicHolidays/" + currentDate.getYear() + "/CA";
            try {
                String webpage = PageReader.readWebPage(theURL);
                currentHolidays = HolidayUtility.getHolidaysFrom(new JSONArray(webpage));
                holidaysYear = currentDate.getYear();
            } catch (IOException | JSONException ie) {
                ie.printStackTrace();
                showError("Failed to get holidays for " + currentDate.getYear());
            }
        }
        if (currentSettings.isMergeHoliday()) {
            MultiEvent.mergeEvents(currentHolidays);
        } else {
            MultiEvent.splitEvents(currentHolidays);
        }
    }

    // MODIFIES: this
    // EFFECTS: shows the setting display
    private void showSettings() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        JPanel listPanel = new JPanel();
        listPanel.setLayout(new BoxLayout(listPanel, BoxLayout.PAGE_AXIS));
        listPanel.add(createLoadSettingPanel(currentSettings));
        listPanel.add(createEventSettingPanel(currentSettings));
        listPanel.add(createColorSettingPanel(currentSettings));
        mainPanel.add(new TitleLabel("Settings"), BorderLayout.PAGE_START);
        mainPanel.add(listPanel, BorderLayout.CENTER);
        changeCentreComponentTo(mainPanel, Display.Settings);
    }

    // EFFECTS: creates the panel for load options
    private JPanel createLoadSettingPanel(Settings settings) {
        JPanel mainPanel = new JPanel();
        mainPanel.setLayout(new BoxLayout(mainPanel, BoxLayout.PAGE_AXIS));
        JCheckBox colorChooser = new JCheckBox("Load on Start");
        colorChooser.setSelected(settings.isLoadOnStart());
        colorChooser.addItemListener(e -> settings.setLoadOnStart(e.getStateChange() == ItemEvent.SELECTED));
        colorChooser.setAlignmentX(Component.LEFT_ALIGNMENT);
        mainPanel.add(createFilePicker(settings));
        mainPanel.add(colorChooser);
        mainPanel.setMaximumSize(mainPanel.getMinimumSize());
        mainPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
        return mainPanel;
    }

    // EFFECTS: create the panel for selecting file
    private JPanel createFilePicker(Settings settings) {
        JPanel mainPanel = new JPanel();
        mainPanel.setLayout(new BoxLayout(mainPanel, BoxLayout.PAGE_AXIS));
        JLabel label = new JLabel("  File Location");
        JTextField fileField = new JTextField(settings.getSaveFile());
        fileField.setMinimumSize(new Dimension(WIDTH, fileField.getMinimumSize().height));
        JButton button = new JButton("Select New Location");
        button.addActionListener(e -> {
            openFilePrompt();
            showSettings();
        });
        fileField.setAlignmentX(Component.LEFT_ALIGNMENT);
        mainPanel.add(label);
        mainPanel.add(fileField);
        mainPanel.add(button);
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
        mainPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
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
        mainPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
        return mainPanel;
    }

    // MODIFIES: this
    // EFFECTS: show help display
    private void showHelp() {
        JPanel mainPanel = new JPanel(new BorderLayout());
        String helpText = "<html>- Use <i>File/Save</i> to save the current schedule<br>"
                + "- Use <i>File/Load</i> to load the saved schedule<br>"
                + "- Use <i>File/Settings</i> to view settings<br>"
                + "&nbsp;&nbsp;- Use <i>Select New Location</i> to find a new location for your calendar<br>"
                + "&nbsp;&nbsp;- The settings file will always be in \"" + Settings.DEFAULT_SETTINGS_FILE + "\"<br>"
                + "- Use <i>Edit/New Event</i> to create a new event<br>"
                + "- Use <i>View/Events</i> to view all events<br>"
                + "- Use <i>View/Calendar</i> to view the calendar<br>"
                + "&nbsp;&nbsp;- Use left/right button below to switch months<br>"
                + "&nbsp;&nbsp;- Click on a date to view the date's events<br>"
                + "- Use <i>View/Holidays</i> to view the holidays for this year";
        JLabel textLabel = new JLabel(helpText);
        textLabel.setVerticalAlignment(SwingConstants.TOP);
        textLabel.setHorizontalAlignment(SwingConstants.LEFT);
        mainPanel.add(new TitleLabel("Help"), BorderLayout.PAGE_START);
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