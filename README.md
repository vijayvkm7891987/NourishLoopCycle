# NourishLoopCycle
An app that can provide solution to hunger, food security and proper utilization of waste food
import 'package:flutter/material.dart';
import 'package:google_generative_ai/google_generative_ai.dart';

void main() {
  runApp(const NourishLoopApp());
}

class NourishLoopApp extends StatelessWidget {
  const NourishLoopApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'NourishLoopCycle',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const DashboardScreen(),
    );
  }
}

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  int _selectedIndex = 0;

  final List<Widget> _pages = [
    const PantryPage(),
    const RecipeAiPage(),
    const CommunityLoopPage(),
    const ImpactPage(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('NourishLoopCycle'),
        backgroundColor: Colors.teal,
        foregroundColor: Colors.white,
      ),
      body: _pages[_selectedIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _selectedIndex,
        selectedItemColor: Colors.teal,
        unselectedItemColor: Colors.grey,
        onTap: (index) => setState(() => _selectedIndex = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.kitchen), label: 'Pantry'),
          BottomNavigationBarItem(icon: Icon(Icons.auto_awesome), label: 'AI Recipes'),
          BottomNavigationBarItem(icon: Icon(Icons.people), label: 'Community'),
          BottomNavigationBarItem(icon: Icon(Icons.bar_chart), label: 'Impact'),
        ],
      ),
    );
  }
}

// --- 1. MY PANTRY PAGE ---
class PantryPage extends StatelessWidget {
  const PantryPage({super.key});

  @override
  Widget build(BuildContext context) {
    final pantryItems = [
      {'name': 'Spinach', 'status': 'Expires Today', 'color': Colors.red},
      {'name': 'Peppers', 'status': 'Expires in 2 Days', 'color': Colors.orange},
      {'name': 'Milk', 'status': 'Expires in 3 Days', 'color': Colors.amber},
    ];

    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text('Needs Attention Today', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
          const SizedBox(height: 10),
          Expanded(
            child: ListView.builder(
              itemCount: pantryItems.length,
              itemBuilder: (context, index) {
                final item = pantryItems[index];
                return Card(
                  child: ListTile(
                    leading: Icon(Icons.fastfood, color: item['color'] as Color),
                    title: Text(item['name'] as String),
                    subtitle: Text(item['status'] as String),
                    trailing: TextButton(
                      child: const Text('Share / Cook'),
                      onPressed: () {},
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

// --- 2. VERTEX AI RECIPE GENERATOR PAGE ---
class RecipeAiPage extends StatefulWidget {
  const RecipeAiPage({super.key});

  @override
  State<RecipeAiPage> createState() => _RecipeAiPageState();
}

class _RecipeAiPageState extends State<RecipeAiPage> {
  final _ingredientsController = TextEditingController(text: 'Spinach, Peppers, Rice');
  String _generatedRecipe = '';
  bool _isLoading = false;

  // Replace with your Vertex AI / Gemini API Key
  static const String _apiKey = 'YOUR_VERTEX_AI_API_KEY';

  Future<void> _generateRecipe() async {
    setState(() {
      _isLoading = true;
      _generatedRecipe = '';
    });

    try {
      final model = GenerativeModel(model: 'gemini-1.5-flash', apiKey: _apiKey);
      final prompt = 'Suggest a simple 15-minute recipe using these expiring ingredients: ${_ingredientsController.text}. Include cooking steps.';
      final response = await model.generateContent([Content.text(prompt)]);

      setState(() {
        _generatedRecipe = response.text ?? 'No recipe generated.';
      });
    } catch (e) {
      setState(() {
        _generatedRecipe = 'Error generating recipe: $e';
      });
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text('Vertex AI Recipe Finder', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
          const SizedBox(height: 10),
          TextField(
            controller: _ingredientsController,
            decoration: const InputDecoration(
              labelText: 'Available Ingredients',
              border: OutlineInputBorder(),
            ),
          ),
          const SizedBox(height: 10),
          ElevatedButton.icon(
            onPressed: _isLoading ? null : _generateRecipe,
            icon: const Icon(Icons.auto_awesome),
            label: const Text('Generate Recipe'),
            style: ElevatedButton.styleFrom(backgroundColor: Colors.teal, foregroundColor: Colors.white),
          ),
          const SizedBox(height: 20),
          if (_isLoading) const CircularProgressIndicator(),
          if (_generatedRecipe.isNotEmpty)
            Card(
              color: Colors.teal.shade50,
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Text(_generatedRecipe, style: const TextStyle(fontSize: 16)),
              ),
            ),
        ],
      ),
    );
  }
}

// --- 3. COMMUNITY LOOP PAGE ---
class CommunityLoopPage extends StatelessWidget {
  const CommunityLoopPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: ListView(
        children: const [
          Text('Community Sharing Loop', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
          SizedBox(height: 10),
          Card(
            child: ListTile(
              leading: Icon(Icons.location_on, color: Colors.teal),
              title: Text('Nearby Food Bank - Station A'),
              subtitle: Text('Collection Window: 2:00 PM - 5:00 PM'),
              trailing: Icon(Icons.chevron_right),
            ),
          ),
          Card(
            child: ListTile(
              leading: Icon(Icons.handshake, color: Colors.teal),
              title: Text('Share Surplus Bread with Neighbour'),
              subtitle: Text('Scheduled for pickup'),
            ),
          ),
        ],
      ),
    );
  }
}

// --- 4. MY IMPACT PAGE ---
class ImpactPage extends StatelessWidget {
  const ImpactPage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Padding(
      padding: EdgeInsets.all(16.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('My Impact', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
          SizedBox(height: 20),
          Card(
            child: ListTile(
              leading: Icon(Icons.eco, color: Colors.green),
              title: Text('12.5 kg Food Saved'),
              subtitle: Text('Prevents waste & reduces carbon emissions'),
            ),
          ),
          Card(
            child: ListTile(
              leading: Icon(Icons.attach_money, color: Colors.green),
              title: Text('Estimated Savings: \$45'),
              subtitle: Text('Based on pantry tracking logs'),
            ),
          ),
        ],
      ),
    );
  }
}
