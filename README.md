import SwiftUI

struct CalendarioView: View {
    @State private var showingAddTask = false // Variabile per mostrare il form di input
    
    var body: some View {
        NavigationView {
            VStack {
                Text("Calendario in arrivo!")
            }
            .navigationTitle("Calendario")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: {
                        showingAddTask.toggle()
                    }) {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showingAddTask) {
                AddTaskView()
            }
        }
    }
}

struct AddTaskView: View {
    @Environment(\.dismiss) var dismiss
    @State private var taskName = ""
    
    var body: some View {
        NavigationView {
            Form {
                TextField("Nome Impegno", text: $taskName)
            }
            .navigationTitle("Aggiungi Impegno")
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Annulla") {
                        dismiss()
                    }
                }
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Salva") {
                        // Azione di salvataggio
                        dismiss()
                    }
                }
            }
        }
    }
}
struct CalendarioView: View {
    @State private var currentDate = Date()
    @State private var showingAddTask = false // Mostra finestra per aggiungere impegni
    @State private var tasks: [Date: [String]] = [:] // Dizionario con impegni per data
    
    private let calendar = Calendar.current
    
    var body: some View {
        NavigationView {
            VStack {
                HStack {
                    Button(action: {
                        currentDate = moveMonth(by: -1, from: currentDate)
                    }) {
                        Image(systemName: "chevron.left")
                    }
                    Spacer()
                    Text("\(monthAndYear(from: currentDate))")
                        .font(.title2)
                    Spacer()
                    Button(action: {
                        currentDate = moveMonth(by: 1, from: currentDate)
                    }) {
                        Image(systemName: "chevron.right")
                    }
                }
                .padding(.horizontal)
                
                CalendarGrid(currentDate: $currentDate, tasks: $tasks)
                    .padding()
                
                Spacer()
            }
            .navigationTitle("Calendario")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: {
                        showingAddTask.toggle()
                    }) {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showingAddTask) {
                AddTaskView(selectedDate: currentDate, tasks: $tasks)
            }
        }
    }
    
    private func monthAndYear(from date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateFormat = "MMMM yyyy"
        return formatter.string(from: date).capitalized
    }
    
    private func moveMonth(by months: Int, from date: Date) -> Date {
        calendar.date(byAdding: .month, value: months, to: date) ?? date
    }
}
struct CalendarGrid: View {
    @Binding var currentDate: Date
    @Binding var tasks: [Date: [String]]
    @State private var selectedDay: Int?
    
    private let calendar = Calendar.current
    private var daysInMonth: [Int] {
        let range = calendar.range(of: .day, in: .month, for: currentDate)!
        return Array(range)
    }
    
    var body: some View {
        LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 7)) {
            ForEach(daysInMonth, id: \.self) { day in
                VStack {
                    Text("\(day)")
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                        .padding()
                        .background(selectedDay == day ? Color.blue.opacity(0.5) : Color.clear)
                        .clipShape(Circle())
                        .onTapGesture {
                            selectedDay = day
                            print("Hai selezionato il giorno \(day)")
                        }
                    
                    // Mostra gli impegni del giorno
                    if let dayTasks = tasks[dateFor(day: day)] {
                        ForEach(dayTasks, id: \.self) { task in
                            Text(task)
                                .font(.caption)
                        }
                    }
                }
            }
        }
    }
    
    private func dateFor(day: Int) -> Date {
        var components = calendar.dateComponents([.year, .month], from: currentDate)
        components.day = day
        return calendar.date(from: components)!
    }
} 
import SwiftUI

struct OrarioView: View {
    @State private var schedule: [String: [String]] = [
        "Lunedì": ["08:00 - Matematica", "10:00 - Italiano"],
        "Martedì": ["09:00 - Filosofia", "11:00 - Scienze"],
        "Mercoledì": ["08:00 - Storia", "10:00 - Inglese"],
        "Giovedì": ["09:00 - Fisica", "11:00 - Educazione fisica"],
        "Venerdì": ["08:00 - Chimica", "10:00 - Arte"]
    ]
    @State private var editingLesson: (day: String, index: Int)? // Lezione in modifica
    @State private var showingAddLesson = false // Mostra finestra per aggiungere lezioni
    
    var body: some View {
        NavigationView {
            List {
                ForEach(schedule.keys.sorted(), id: \.self) { day in
                    Section(header: Text(day)) {
                        ForEach(schedule[day]?.indices ?? 0..<0, id: \.self) { index in
                            HStack {
                                Text(schedule[day]?[index] ?? "")
                                Spacer()
                                Button(action: {
                                    editingLesson = (day, index) // Segna la lezione in modifica
                                }) {
                                    Image(systemName: "pencil")
                                        .foregroundColor(.blue)
                                }
                            }
                        }
                        .onDelete { offsets in
                            deleteLesson(from: day, at: offsets)
                        }
                    }
                }
            }
            .navigationTitle("Orario")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: {
                        showingAddLesson.toggle()
                    }) {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showingAddLesson) {
                AddLessonView(schedule: $schedule)
            }
            .sheet(item: $editingLesson) { lesson in
                EditLessonView(schedule: $schedule, day: lesson.day, index: lesson.index)
            }
        }
    }
    
    // Funzione per eliminare una lezione
    private func deleteLesson(from day: String, at offsets: IndexSet) {
        schedule[day]?.remove(atOffsets: offsets)
    }
}
import SwiftUI

struct EditLessonView: View {
    @Environment(\.presentationMode) var presentationMode
    @Binding var schedule: [String: [String]]
    
    let day: String
    let index: Int
    
    @State private var lessonTime: String = ""
    @State private var lessonName: String = ""
    
    var body: some View {
        NavigationView {
            Form {
                TextField("Orario (es. 08:00)", text: $lessonTime)
                TextField("Nome lezione", text: $lessonName)
            }
            .navigationTitle("Modifica Lezione")
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Annulla") {
                        presentationMode.wrappedValue.dismiss()
                    }
                }
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Salva") {
                        saveChanges()
                        presentationMode.wrappedValue.dismiss()
                    }
                }
            }
            .onAppear {
                loadCurrentLesson()
            }
        }
    }
    
    private func loadCurrentLesson() {
        let currentLesson = schedule[day]?[index] ?? ""
        let components = currentLesson.split(separator: "-", maxSplits: 1).map { $0.trimmingCharacters(in: .whitespaces) }
        if components.count == 2 {
            lessonTime = components[0]
            lessonName = components[1]
        }
    }
    
    private func saveChanges() {
        guard !lessonTime.isEmpty, !lessonName.isEmpty else { return }
        schedule[day]?[index] = "\(lessonTime) - \(lessonName)"
    }
}
struct OrarioTableView: View {
    @Binding var schedule: [String: [String]]
    
    var body: some View {
        ScrollView {
            LazyVGrid(columns: [
                GridItem(.flexible()), // Colonna giorno
                GridItem(.flexible())  // Colonna lezioni
            ], spacing: 10) {
                // Intestazioni
                Text("Giorno")
                    .font(.headline)
                Text("Lezioni")
                    .font(.headline)
                
                // Dati
                ForEach(schedule.keys.sorted(), id: \.self) { day in
                    Text(day)
                        .font(.subheadline)
                        .frame(maxWidth: .infinity, alignment: .leading)
                    
                    VStack(alignment: .leading) {
                        ForEach(schedule[day] ?? [], id: \.self) { lesson in
                            Text(lesson)
                                .font(.subheadline)
                        }
                    }
                }
            }
            .padding()
        }
        .navigationTitle("Tabella Orario")
    }
}
import SwiftUI

struct VotiView: View {
    @State private var grades: [String: [(date: Date, grade: Double)]] = [
        "Matematica": [(date: Date(), grade: 7.5), (date: Date().addingTimeInterval(-86400 * 5), grade: 8.0)],
        "Italiano": [(date: Date(), grade: 6.0)],
        "Filosofia": [(date: Date(), grade: 9.0)]
    ]
    
    @State private var showingAddGrade = false
    
    var body: some View {
        NavigationView {
            List {
                ForEach(grades.keys.sorted(), id: \.self) { subject in
                    Section(header: Text(subject)) {
                        ForEach(grades[subject] ?? [], id: \.date) { grade in
                            VStack(alignment: .leading) {
                                Text("Data: \(formattedDate(grade.date))")
                                Text("Voto: \(grade.grade, specifier: "%.1f")")
                            }
                        }
                    }
                }
            }
            .navigationTitle("Voti")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: {
                        showingAddGrade.toggle()
                    }) {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showingAddGrade) {
                AddGradeView(grades: $grades)
            }
        }
    }
    
    private func formattedDate(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .short
        return formatter.string(from: date)
    }
}
import SwiftUI

struct AddGradeView: View {
    @Environment(\.presentationMode) var presentationMode
    @Binding var grades: [String: [(date: Date, grade: Double)]]
    
    @State private var selectedSubject = "Matematica"
    @State private var grade: Double?
    @State private var date = Date()
    
    let subjects = ["Matematica", "Italiano", "Filosofia", "Scienze", "Storia"]
    
    var body: some View {
        NavigationView {
            Form {
                Picker("Materia", selection: $selectedSubject) {
                    ForEach(subjects, id: \.self) { subject in
                        Text(subject)
                    }
                }
                
                DatePicker("Data", selection: $date, displayedComponents: .date)
                
                TextField("Voto", value: $grade, format: .number)
                    .keyboardType(.decimalPad)
            }
            .navigationTitle("Aggiungi Voto")
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Annulla") {
                        presentationMode.wrappedValue.dismiss()
                    }
                }
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Salva") {
                        saveGrade()
                        presentationMode.wrappedValue.dismiss()
                    }
                }
            }
        }
    }
    
    private func saveGrade() {
        guard let grade = grade else { return }
        let newGrade = (date: date, grade: grade)
        grades[selectedSubject, default: []].append(newGrade)
    }
}
import SwiftUI

struct GradeChartView: View {
    var grades: [(date: Date, grade: Double)]
    
    var body: some View {
        VStack {
            Text("Andamento Voti")
                .font(.title)
            LineChartView(data: grades.map { $0.grade }, title: "Voti")
                .padding()
        }
    }
}
import SwiftUI

struct MainView: View {
    @State private var showingAddCommitment = false
    @State private var showingAddGrade = false
    @State private var commitments: [String: [String]] = [:]
    @State private var grades: [String: [(date: Date, grade: Double)]] = [:]
    
    var body: some View {
        NavigationView {
            VStack {
                // Visualizza il calendario
                CalendarView()
                    .padding()
                
                // Visualizza l'orario
                OrarioView()
                    .padding()
                
                // Visualizza i voti
                VotiView()
                    .padding()
                
                Spacer()
            }
            .navigationTitle("Gestione Scolastica")
            .toolbar {
                ToolbarItemGroup(placement: .navigationBarTrailing) {
                    Button(action: {
                        showingAddCommitment.toggle()
                    }) {
                        Image(systemName: "plus")
                        Text("Aggiungi Impegno")
                    }
                    
                    Button(action: {
                        showingAddGrade.toggle()
                    }) {
                        Image(systemName: "plus")
                        Text("Aggiungi Voto")
                    }
                }
            }
            .sheet(isPresented: $showingAddCommitment) {
                AddCommitmentView(commitments: $commitments)
            }
            .sheet(isPresented: $showingAddGrade) {
                AddGradeView(grades: $grades)
            }
        }
    }
}
import SwiftUI

struct AddCommitmentView: View {
    @Environment(\.presentationMode) var presentationMode
    @Binding var commitments: [String: [String]]
    
    @State private var selectedDate = Date()
    @State private var commitmentText = ""
    
    var body: some View {
        NavigationView {
            Form {
                DatePicker("Data", selection: $selectedDate, displayedComponents: .date)
                TextField("Impegno", text: $commitmentText)
            }
            .navigationTitle("Aggiungi Impegno")
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("Annulla") {
                        presentationMode.wrappedValue.dismiss()
                    }
                }
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Salva") {
                        saveCommitment()
                        presentationMode.wrappedValue.dismiss()
                    }
                }
            }
        }
    }
    
    private func saveCommitment() {
        let day = formattedDate(selectedDate)
        commitments[day, default: []].append(commitmentText)
    }
    
    private func formattedDate(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .short
        return formatter.string(from: date)
    }
}
    
