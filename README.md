#include <iostream>
#include <cmath>
#include <fstream>
#include <ctime>

using namespace std;

class StaticBalanceCalculator {
private:
    const double g;
    const double PI;

public:
    struct BalanceResult {
        double lift_force;           // 升力（垂直方向）
        double drag_force;           // 阻力（水平方向）
        double left_tension;
        double right_tension;
        double left_lift_fixed;      // 左侧固定端升力
        double left_lift_pulley;     // 左侧滑轮端升力
        double right_lift_fixed;     // 右侧固定端升力
        double right_lift_pulley;    // 右侧滑轮端升力
        double left_drag_fixed;      // 左侧固定端阻力
        double left_drag_pulley;     // 左侧滑轮端阻力
        double right_drag_fixed;     // 右侧固定端阻力
        double right_drag_pulley;    // 右侧滑轮端阻力
        double wing_weight_N;

        BalanceResult() : lift_force(0), drag_force(0),
                         left_tension(0), right_tension(0),
                         left_lift_fixed(0), left_lift_pulley(0),
                         right_lift_fixed(0), right_lift_pulley(0),
                         left_drag_fixed(0), left_drag_pulley(0),
                         right_drag_fixed(0), right_drag_pulley(0),
                         wing_weight_N(0) {}
    };

    StaticBalanceCalculator() : g(9.81), PI(3.14159265358979323846) {}

    void getUserInput(double& left_mass_g, double& right_mass_g,
                      double& left_fixed_angle, double& left_pulley_angle,
                      double& right_fixed_angle, double& right_pulley_angle,
                      double& wing_weight_g)
    {
        cout << "=== 静力平衡升力计算 ===" << endl;

        cout << "输入左侧量杯质量 (g): ";
        cin >> left_mass_g;
        
        cout << "输入右侧量杯质量 (g): ";
        cin >> right_mass_g;

        cout << "\n左侧角度:" << endl;
        cout << "固定端角度 α1 (°): ";
        cin >> left_fixed_angle;
        cout << "滑轮端角度 α2 (°): ";
        cin >> left_pulley_angle;

        cout << "\n右侧角度:" << endl;
        cout << "固定端角度 α1 (°): ";
        cin >> right_fixed_angle;
        cout << "滑轮端角度 α2 (°): ";
        cin >> right_pulley_angle;

        cout << "\n输入机翼重量 (g): ";
        cin >> wing_weight_g;
    }

    BalanceResult calculateBalance(double left_mass_g, double right_mass_g,
                                   double left_fixed_angle, double left_pulley_angle,
                                   double right_fixed_angle, double right_pulley_angle,
                                   double wing_weight_g)
    {
        BalanceResult result;

        result.wing_weight_N = (wing_weight_g / 1000.0) * g;
        result.left_tension = (left_mass_g / 1000.0) * g;
        result.right_tension = (right_mass_g / 1000.0) * g;

        // 计算角度（转换为弧度）
        double left_angle1_rad = left_fixed_angle * PI / 180.0;
        double left_angle2_rad = (left_fixed_angle + left_pulley_angle) * PI / 180.0;
        double right_angle1_rad = right_fixed_angle * PI / 180.0;
        double right_angle2_rad = (right_fixed_angle + right_pulley_angle) * PI / 180.0;
        
        // 计算左侧升力（垂直分力）
        result.left_lift_fixed = result.left_tension * sin(left_angle1_rad);
        result.left_lift_pulley = result.left_tension * sin(left_angle2_rad);
        
        // 计算左侧阻力（水平分力）
        result.left_drag_fixed = result.left_tension * cos(left_angle1_rad);
        result.left_drag_pulley = result.left_tension * cos(left_angle2_rad);
        
        // 计算右侧升力（垂直分力）
        result.right_lift_fixed = result.right_tension * sin(right_angle1_rad);
        result.right_lift_pulley = result.right_tension * sin(right_angle2_rad);
        
        // 计算右侧阻力（水平分力）
        result.right_drag_fixed = result.right_tension * cos(right_angle1_rad);
        result.right_drag_pulley = result.right_tension * cos(right_angle2_rad);
        
        // 计算总升力（垂直分力相加）
        double total_lift_from_ropes = 
            (result.left_lift_fixed + result.left_lift_pulley) +
            (result.right_lift_fixed + result.right_lift_pulley);
        
        // 计算净水平分力（左右水平分力相减，因为方向相反）
        double left_total_horizontal = result.left_drag_fixed + result.left_drag_pulley;
        double right_total_horizontal = result.right_drag_fixed + result.right_drag_pulley;
        double net_horizontal_force = left_total_horizontal - right_total_horizontal;
        
        // 在静力平衡条件下，空气动力升力 = 绳子提供的总升力 - 机翼重量
        result.lift_force = total_lift_from_ropes - result.wing_weight_N;
        
        // 阻力就是绳子提供的净水平分力
        result.drag_force = net_horizontal_force;

        return result;
    }

    void displayResults(const BalanceResult& result,
                        double left_mass_g, double right_mass_g,
                        double left_fixed_angle, double left_pulley_angle,
                        double right_fixed_angle, double right_pulley_angle,
                        double wing_weight_g)
    {
        cout.setf(ios::fixed);
        cout.precision(4);

        cout << "\n=== 计算结果 ===" << endl;

        cout << "左杯质量: " << left_mass_g << " g"
             << "  右杯质量: " << right_mass_g << " g" << endl;

        cout << "左角度 α1=" << left_fixed_angle 
             << "°, α2=" << left_pulley_angle << "°" << endl;
             
        cout << "右角度 α1=" << right_fixed_angle 
             << "°, α2=" << right_pulley_angle << "°" << endl;
             
        cout << "机翼重量: " << wing_weight_g << " g  ("
             << result.wing_weight_N << " N)" << endl << endl;

        cout << "左侧分力:" << endl;
        cout << "  固定端 垂直=" << result.left_lift_fixed 
             << " 水平=" << result.left_drag_fixed << endl;
        cout << "  滑轮端 垂直=" << result.left_lift_pulley 
             << " 水平=" << result.left_drag_pulley << endl;
             
        cout << "右侧分力:" << endl;
        cout << "  固定端 垂直=" << result.right_lift_fixed 
             << " 水平=" << result.right_drag_fixed << endl;
        cout << "  滑轮端 垂直=" << result.right_lift_pulley 
             << " 水平=" << result.right_drag_pulley << endl;

        double total_vertical = 
            result.left_lift_fixed + result.left_lift_pulley +
            result.right_lift_fixed + result.right_lift_pulley;
            
        double left_total_horizontal = 
            result.left_drag_fixed + result.left_drag_pulley;
        double right_total_horizontal = 
            result.right_drag_fixed + result.right_drag_pulley;

        cout << endl << "总垂直分力: " << total_vertical << " N" << endl;
        cout << "机翼重量: " << result.wing_weight_N << " N" << endl;
        cout << "升力 = " << result.lift_force << " N" << endl;
        
        // 只显示净水平分力，不显示总水平分力
        cout << "净水平分力 (阻力) = " << result.drag_force << " N" << endl;

        // 静力平衡验证
        cout << endl << "=== 静力平衡验证 ===" << endl;
        if (abs(result.drag_force) < 0.001) {
            cout << "✓ 水平方向力平衡: 净水平分力 ≈ 0" << endl;
        } else {
            cout << "⚠ 水平方向力不平衡: 净水平分力 = " << result.drag_force << " N" << endl;
        }
        
        if (abs(result.lift_force) < 0.001) {
            cout << "✓ 垂直方向力平衡: 净升力 ≈ 0" << endl;
        } else {
            cout << "⚠ 垂直方向力不平衡: 净升力 = " << result.lift_force << " N" << endl;
        }
    }

    void saveToFile(double left_mass_g, double right_mass_g,
                    double left_fixed_angle, double left_pulley_angle,
                    double right_fixed_angle, double right_pulley_angle,
                    double wing_weight_g,
                    const BalanceResult& result,
                    const string& filename)
    {
        ofstream file(filename.c_str(), ios::app);
        if (!file.is_open()) {
            cout << "无法写入档案。" << endl;
            return;
        }

        file.setf(ios::fixed);
        file.precision(4);

        time_t now = time(0);
        char* dt = ctime(&now);

        file << "记录时间: " << dt;

        file << "质量(g): 左=" << left_mass_g << ", 右=" << right_mass_g << " | ";

        file << "左角度: α1=" << left_fixed_angle 
             << ", α2=" << left_pulley_angle << " | ";
             
        file << "右角度: α1=" << right_fixed_angle 
             << ", α2=" << right_pulley_angle << " | ";

        file << "机翼重: " << wing_weight_g << "g | ";
        file << "升力: " << result.lift_force << " N | ";
        file << "阻力: " << result.drag_force << " N" << endl;

        file << "-------------------------------------" << endl;

        cout << "资料已写入 " << filename << endl;
    }
};

int main()
{
    StaticBalanceCalculator calc;
    char again;

    do {
        double left_mass_g, right_mass_g;
        double left_fixed_angle, left_pulley_angle;
        double right_fixed_angle, right_pulley_angle;
        double wing_weight_g;

        calc.getUserInput(left_mass_g, right_mass_g, 
                         left_fixed_angle, left_pulley_angle,
                         right_fixed_angle, right_pulley_angle,
                         wing_weight_g);

        StaticBalanceCalculator::BalanceResult result =
            calc.calculateBalance(left_mass_g, right_mass_g,
                                 left_fixed_angle, left_pulley_angle,
                                 right_fixed_angle, right_pulley_angle,
                                 wing_weight_g);

        calc.displayResults(result, left_mass_g, right_mass_g,
                           left_fixed_angle, left_pulley_angle,
                           right_fixed_angle, right_pulley_angle,
                           wing_weight_g);

        calc.saveToFile(left_mass_g, right_mass_g,
                       left_fixed_angle, left_pulley_angle,
                       right_fixed_angle, right_pulley_angle,
                       wing_weight_g, result, "static_balance_data.txt");

        cout << endl << "再计算一次? (y/n): ";
        cin >> again;

    } while (again == 'y' || again == 'Y');

    cout << "感谢使用静力平衡计算器！" << endl;
    return 0;
}
